---
layout: post
title:  "Lessons from 20 Years of Development"
date:   2016-09-29
categories: software engineering, development
---

I've been writing software professionally for most of the last two decades. In this time I've worked just about everywhere: on government contracts, at healthcare companies, in universities, at very big companies (more than 100K employees) and very small ones (fewer than 10 employees). I've written clients, servers, web applications, utilities, plugins, and frameworks. 

Here are some of the things I've learned over those twenty years. They're my opinions, but they've served me well.

**Don't use state, and make state immutable wherever possible.** Immutable state is not an unalloyed good. Used thoughtlessly, it leads to very inefficient programs. However, like static typing, it eliminates a large class of bugs. Many bugs are caused by the program entering an unexpected state; these are not only common but difficult to reproduce as "how did this value (state) get into memory" is a difficult question to answer, even if you're good with data breakpoints. In contrast, software engineering approaches which use less state sometimes make finding the source of the invalid state as easy as walking the callstack. 

**Don't optimize prematurely.** In school, and in interviews, we're all taught to think very hard about the big-O complexity of our algorithms, for it's often the case that the one-off program you wrote to handle 10 items (for which an N^2 algorithm only costs 100) now needs to handle 10,000 (for which it now costs 10,000,000). However, in most real-world applications, inefficiencies are only rarely due to poorly written algorithms. They are more often due to architectural decisions, such as marshaling data between too many layers, using heavyweight components to do simple jobs, and the like. It's so difficult to know for sure where your performance problems lie that it's usually best to get the system working in its simplest form and then using a profiler to figure out where the actual performance problems lie.

**Use statically typed languages.** It is sometimes faster, and sometimes easier, to code without type information. However, statically typed languages push an entire class of bugs from runtime (where my users see them) to compile time (where I see them). This is invaluable. Statically typed languages also tremendously improve the development experience by allowing for the use of refactoring tools and improving autocompletion/intellisense (as the editor itself knows what you can do with an object of a known type). It is for this reason that I'm a big fan of projects like [TypeScript](https://www.typescriptlang.org/).

**Be declarative, not imperative.** It's easier to screw up instructions than information. If you can encode a problem as an algorithm acting on data instead of a more complicated algorithm, you'll rarely regret it.

**Code for the maintainer.** Some people leave copious comments. Some people think they can write self-documenting code. I think they're wrong, but that's another story. No matter which way you slice it, though, code with an eye towards making the maintainer's life easy. Code is read much more often than it is written. Code simply. Don't lay traps. Keep state local so that it's obvious what will be affect when code changes. That maintainer you're coding for is probably yourself. 

**Keep modules small and loosely coupled.** You'd think this would go without saying, but it doesn't. Complexity will kill you. You can only handle so much of it, so split it into smaller pieces until your brain can understand each piece. If you're me, those pieces are pretty small.

**Think first, code later**. Just twenty minutes sketching out the outline of what I'm doing helps me think about what I'm building end-to-end. If I can't at least walk through how it's going to work in my head, I don't often start coding, because I'm liable to waste time going down dead ends. 

**Start coding as soon as you can**. Some people plan in a very detailed way before they start coding. I'm of the opinion that this is often a waste of time. At a sufficiently low level of detail, you may as well be coding. Once you have a rough picture of the system in your head and have the major technical decisions made, there is no substitute for actually writing code. It's often easier to let your code tell you what you need to get the system working than it is to guess exhaustively ahead of time.

**Be simple, not clever.** An engineer wiser than myself (Brian Kernigham) once said:

> Everyone knows that debugging is twice as hard as writing a program in the first place. So if you're as clever as you can be when you write it, how will you ever debug it?

Having authored several clever systems which later turned out to be nightmarish to debug, I've found this advice rings true. Use simple mechanisms you understand thoroughly. If you must use a new mechanism, or invent one, make sure you're choosing the simplest one for the job. 

**Self-test ruthlessly.** An unfortunate truth: you know your system too well to test it correctly. Your users, or your QA team if you have one, are going to use your software in ways you did not anticipate or think about. This, however, does not absolve you of the responsibility to ruthlessly test your own work as best you can before you let it out in the wild; in fact, while your status as the system's designer gives you some testing blind spots, it also gives you some insight your QA team and users will never have about where the true edge cases lie. Be pessimistic. Try to put yourself in your users' shoes. Develop a reputation for delivering software that works the first time.

**Invest in good tools.** It amazes me how many engineers use substandard tooling to develop software when the world is teeming with powerful, extensible editors and IDEs. They all have strengths and weaknesses; the important thing is to pick one and ruthlessly configure it until it does exactly what you need. My editor of choice is Vim, but there are dozens of others that can be great if you just give them a little time and attention. At a minimum, your editor should highlight your syntax, offer auto-completion of properties and methods, and give you warnings about improper syntax and usage without requiring a compile pass.

**Learn makefile syntax.** GNU Make is a remarkably powerful tool, and you can use it to automate the generation of *almost anything* from *almost anything else*, in a declarative way. Once mastered, it can efficiently produce websites, documents, and all manner of other deliverables. It works with any command you can execute at a shell. It's an essential part of a reproducible workflow. 

**Use plain text files and tools.** Plain text is the *lingua franca* of the software engineer. It can be consumed by decades-old tools and will be readable decades from now. Plain text can be version controlled, and takes up little space. It's human-readable. It's easy to copy, paste, format, share, and remix. You won't regret having data in plain text.

**Write tests, but not religiously.** Tests are valuable. They can give you great confidence when making changes, and reduce the risk of regressions. They are also a wonderful forcing function: writing a test for your code forces you to think about it thoroughly and critically. However, tests also increase the cost of changes to your code, because you must also write and change tests. I've seen buggy, slow software with plenty of tests, and fast, accurate software with almost none. Tests are not a silver bullet, and the easiest tests to write are often the least useful. Use them, but use them cautiously.

**First, make it work end-to-end.** There are lots of difficult decisions and complicated pieces in most software projects. It's very easy to get distracted by one of these before your software does anything useful, and to continue to be distracted because it's easier to focus on a small, known problem than tackle a large, unknown one. However, until your software does *something*, until it works end-to-end for its simplest use case, there's a good chance there's something big you haven't thought of yet. Get it working before you worry about the details.

**Learn constantly.** Nowhere is the old adage "Change is the only constant" more true than in software engineering. Software engineers are in the unique position of being the primary fashioners of their own tools, and boy, do they make a lot of them. New platforms, paradigms, systems, and languages spring up every few weeks. Spend some time learning about these and playing with them if you have time, but don't make the mistake of thinking that just because it's new means it's better; many new things fizzle, whereas old things have by definition stood the test of time.

**Add a new tool to your personal toolkit.** Every now and again, think about the way you work. Could something be better? Are you, for instance, spending a lot of time switching among lots of terminal windows? Or is managing time and tasks difficult for you? Or are you continually frustrated by your mail program? All of these problems, and many more, have dozens of clever software solutions you probably haven't have heard of but could easily find with some web searching. Regularly evaluate your personal toolkit with a critical eye and add new tools to it. If there's nothing you can solve with off the shelf software, what about writing a script of your own? (Some developers have the opposite problem, in which we obsess over our personal toolkit at the expense of actually building stuff with it. I certainly don't know any developers like *that*.)

**Always think about the end goal.** Solving technical problems is so much fun for most of us that we often forget to think about whether they are worth solving. The relationship between the technical difficulty of a problem and the value of solving it is almost non-existent. Always ask yourself whether what you're doing is going to make a big difference to someone using your software. Work on the things that matter to your users. 

**Know when to buy and when to build.** Whichever one you choose, it will be more expensive than you thought, and not as good. Choose wisely.

Of course, this is all about the act of engineering itself -- which is only a small portion of your job as a software engineer. I hope, however, that it's helpful. 


