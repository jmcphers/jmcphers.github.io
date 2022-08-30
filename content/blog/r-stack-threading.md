---
title: "The Case of R's Stack Overflow"
date: 2022-08-30
---

## The Problem

So, there I was, working on a project that embeds R in a Rust program using [extendr](https://github.com/extendr/extendr), and everything was going great, until I tried to run the program in an aarch64 Linux environment. R emitted a whole bunch of these errors and wouldn't start up.

```
Error: C stack usage  1474053204 is too close to the limit
Error: C stack usage  1474052564 is too close to the limit
Fatal error: unable to initialize the JIT
```

I thought this would be relatively easy to fix. This blog post is about how wrong I was.

## What's a "C Stack"?

When you allocate memory in a computer program, it usually gets allocated in one of two places -- the *stack* or the *heap*. The *stack* is usually used for things like local variables that wink in and out of existence as functions are executed; the *heap* is usually used to allocate memory for longer-lived data that is manipulated over time. They grow in different directions.

```goat
.-------+-------------------------+------.
| Stack +--->                 <---+ Heap |
'-------+-------------------------+------'
```
The "C Stack" is just the stack created by programs written in the computer language C, which includes R itself.

## What's a "C Stack Limit?

You might have heard of a **stack overflow**. It's probably the [most popular website for programming questions](https://www.stackoverflow.com). It's also a condition that occurs when you try to put too much data onto the stack. If you completely exhaust the stack, memory allocations start failing, and most programs will crash (losing all your data or worse) if they can't allocate memory. You *absolutely don't* want a stack overflow, so R regularly checks stack memory usage and starts sounding the alarm when things get too close.

Modern computers have a lot of memory, and they have very generous stack sizes, so you have to be trying pretty hard to overflow them. By far the most common cause of stack overflows is infinite recursion: if you have a function that puts data on the stack, then calls itself, then you can create a loop that fills up the stack quickly. This function, for example, will overflow the stack:

```c
void overflow() {
    int data[1024];
    overflow();
}
```

The "too close to the limit" message is telling us that R thinks there's too much data on the stack, and a stack overflow is imminent.

## Checking the Stack

Since a stack overflow is most often caused by infinite recursion, my first guess was that some function was calling itself a whole bunch of times. All I had to do was set a breakpoint where the stack overflow was being reported, and look at the callstack to see what it was.

So I set a breakpoint at `R_SignalCStackOverflow` and waited. Here's the callstack that came back:

```
#0  R_SignalCStackOverflow (usage=188041156) at errors.c:100
#1  0x0000fffff7c4c9b8 in Rf_eval (e=0xfffff005bea0, rho=rho@entry=0xfffff003e940) at eval.c:735
#2  0x0000fffff7c7db88 in R_ReplFile (fp=fp@entry=0xfffff00010e0, rho=0xfffff003e940) at main.c:99
#3  0x0000fffff7c7ea2c in setup_Rmainloop () at main.c:976
#4  0x0000fffff7c7fddc in Rf_mainloop () at main.c:1145
```

So much for the "runaway recursion" theory. In fact, the stack is overflowing with *almost nothing on the stack*. What in the world? 

## Detecting an Overflow

Okay, so R thinks the stack is overflowing, but it also looks like there's almost nothing on the stack. Could R just be getting it wrong? Under what conditions, exactly, does R think a stack overflow has occurred? To answer that question, it was time to check the R source code. Here's how R checks for a stack overflow (in [errors.c](https://github.com/wch/r-source/blob/b05fee3972a7541945b7d36655469c0b05b6565f/src/main/errors.c#L114-L122)):

```c
void (R_CheckStack)(void)
{
    int dummy;
    intptr_t usage = R_CStackDir * (R_CStackStart - (uintptr_t)&dummy);

    /* printf("usage %ld\n", usage); */
    if(R_CStackLimit != -1 && usage > ((intptr_t) R_CStackLimit))
	R_SignalCStackOverflow(usage);
}
```

This uses some clever pointer arithmetic to see the size of the stack. In pseudocode:

1. Create a variable called `dummy` on the stack.
2. Get the memory address of the variable.
3. Subtract the address from the starting memory address of the stack.
4. The difference between these two values must be the size of the stack. If that size is larger than the stack limit, we have an overflow.

The next step is to see what all those values of each of those variables actually are at runtime, so we can see where things are going wrong.

## Stepping Through the Code

Unfortunately, even with a debug build of R, this function is [inlined](https://en.wikipedia.org/wiki/Inline_function) and can't be stepped through with a debugger. So we're going to have to get a little closer to the metal and step through the assembly code to see what's happening. Our goal is to see why, exactly, R thinks there's a stack overflow here. 

To step through the assembly, I did the following:

- Ran the code under `gdb`.
- Set a breakpoint in the function that signals the overflow, `Rf_eval`.
- When I reached the inlined function, issued the following commands to start stepping by instructions. The first one turns on display of the instruction; the second one steps by instructions (instead of lines).

```
(gdb) display/i $pc
(gdb) stepi
```

I'm going to skip past the boring parts. Here is the `sub` assembly instruction that performs the subtraction in the above method. It is operating on the `x0` and `x4` registers, so let's take a look at those registers (`i r` is short for `info register`). 

```
=> 0xfffff7c4c2bc <Rf_eval+296>:        sub     x0, x0, x4
(gdb) i r x0
x0             0x1000000000000     281474976710656
(gdb) i r x4
x4             0xfffff4cab83c      281474788669500
```

These must correspond to `R_CStackStart` and `&dummy`, respectively. In fact, we can ask gdb to evaluate those symbols directly:

```
(gdb) p R_CStackLimit
$5 = 7969177
(gdb) p R_CStackStart - (intptr_t)&dummy
$4 = 135881456
```

There's no math or overflow issues here -- there is legitimately a huge gulf, about 135 megabytes, between `R_CStackStart` and the place where new variables are allocated on the stack, and this exceeds the stack limit of 8 megabytes by ... well, about 127 megabytes. 

## Questioning Assumptions

We are left with two possibilities here:

1. There **is** a legitimate stack overflow. It can't be infinite recursion or even excessive nesting given the call stack, so something somewhere is allocating an enormous object on the stack, and R doesn't want to put anything else on it.
2. There **is not** a legitimate stack overflow; one or more of the assumptions that is made by R is invalid.

Because there's so little on the stack, the latter seems more likely. So let's dig a little deeper into that one.

## Where Does R Think the Stack Begins?

Of all of the assumptions made by R, the most questionable is the *starting point of the stack*. It's kind of hard to be wrong about the address of the `dummy` variable used in this example-- we'd sure be in a world of hurt if the `&` operator didn't work correctly. What about the stack starting point?

The code that R uses to determine this is easily found. I won't paste the whole thing here, but here's the first part (from [system.c](https://github.com/wch/r-source/blob/7af4ea608b246b4f796cd7a259b7509d932f76b1/src/unix/system.c#L239)):

```c
int Rf_initialize_R(int ac, char **av)
{
    /* lines omitted */
	R_CStackStart = (uintptr_t) __libc_stack_end;
}
```

This `__libc_stack_end` constant's value is close to where R believes the stack begins. 

## Where Does the Stack *Actually* Begin?

At this point I'm going to have to pull back the curtain a little and reveal that this program is multi-threaded. Every thread has its own stack. So since the question we're trying to answer is "where does the stack start for the thread executing R code?", we'll set a breakpoint at `start_thread` (part of [pthreads](https://en.wikipedia.org/wiki/Pthreads)). 

Then we'll print the contents of the [SP (Stack Pointer) register](https://developer.arm.com/documentation/dui0801/a/Overview-of-AArch64-state/Stack-Pointer-register). This will tell us the approximate location of the stack before anything is allocated on it, and it *should* line up with where R thinks the stack starts.

In fact, let's print this register on two different threads -- the main thread (Thread 1) and the thread executing R code (Thread 2). We'll use gdb's `thread` command to switch thread context, and `i r sp` to read the value of the stack pointer register.


```
(gdb) thread 1
[Switching to thread 1 (Thread 0xfffff7a69020 (LWP 76385))]
(gdb) i r sp
sp             0xffffffff7440      0xffffffff7440

(gdb) thread 2
[Switching to thread 2 (Thread 0xfffff4cbec60 (LWP 76388))]
(gdb) i r sp
sp             0xfffff4cbdd90      0xfffff4cbdd90
```

If you've been keeping up thus far, it is probably obvious that we've found the culprit.

## What's Going On Here?

The problem, in short, is that R is *wrong* about where the stack starts. It is making the assumption that it's running on the main thread, and consequently it's setting `R_CStackStart` to the beginning of the stack for Thread 1.

However, we are initializing R on Thread 2. So when `R_CheckStack` allocates `dummy`, it's doing it on Thread 2. Thread 2's stack starts a long way from Thread 1's stack. 

```goat
&dummy (expected)
0xffffffff7440
 |   
 v
.--------------------------+------------------------.
| Thread 1 Stack           | Thread 2 Stack         |
'--------------------------+------------------------'
^                           ^
|                           |
R_CStackStart               &dummy (actual)
0x1000000000000             0xfffff4cab83c
```

No wonder it thinks there's an overflow -- these values are indeed very far apart.

It turns out this is a [known issue with R in multi-threaded programs](https://cran.r-project.org/doc/manuals/R-exts.html#Threading-issues) (thanks to [Kevin Ushey](https://kevinushey.github.io/) for pointing me here). 

> Embedded R is designed to be run in the main thread, and all the testing is done in that context. There is a potential issue with the stack-checking mechanism where threads are involved.

"Potential issue" -- well, in our case, it's an actual issue.

## The Workaround

The resource linked above suggests the appropriate workaround:

> Stack-checking can be disabled by setting `R_CStackLimit = (uintptr_t)-1` immediately after `Rf_initialize_R` is called, but it is better to if possible set appropriate values. (What these are and how to determine them are OS-specific, and the stack size limit may differ for secondary threads. If you have a choice of stack size, at least 10Mb is recommended.)

This does, in fact, fix the problem. Case closed!

## Notes & Lessons

I wish I could say that this blog post summed up my investigation into this issue, but it benefits enormously from hindsight. It outlines the direct path to a solution and omits the dead ends. It took me *four or five attempts* at solving the problem in other ways before I started going down the road of actually trying to dig into the code and physical stack sizes. Debugging is hard.

This kind of analysis is possible only because I had access to the source code of R itself, so I could see how it was working, and good documentation from the R developers describing the problem and how to work around it. Another great reminder to **bet on open source software** -- when something goes wrong, it is absolutely invaluable to be able to read the code to see why.


