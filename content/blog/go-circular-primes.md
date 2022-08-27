---
layout: post
title:  "Finding Circular Primes in Go"
date:   2015-09-03 12:32:32
categories: programming
---

Recently a friend of mine posted the following programming challenge, which
he'd received as an interview question.

> A number is called a circular prime if it is prime and all of its rotations are primes. 
>
> For example the number 197 has two rotations: 971, and 719. Both of them are prime. 
>
> There are thirteen such primes below 100: 2, 3, 5, 7, 11, 13, 17, 31, 37, 71, 73, 79, and 97.
>
> How many circular primes are there below N? 
> 1 <= N <= 1000000
>
> Test Cases:
>
> There are 0 circular primes below 1. 
> There are 4 circular primes below 10. 
> There are 13 circular primes below 100. 
> There are 25 circular primes below 1000.

Several solutions were posted but none were very fast. To solve the problem efficiently, it's helpful to break it down into smaller pieces:

1. How can we efficiently discover all the primes beneath 1,000,000? 
2. How can we efficiently test a prime to see if it's *circular*?

# Finding Primes

The most common method for finding primes is using trial division. To determine if a number *x* is prime, you divide it by all the numbers from *2 - sqrt(x)*. If it divides evenly by any of these, the number is not prime.

This isn't a bad way to test a *single* number for primality (though there are of course much faster, and more sophisticated, tests). However, it's a bad way to generate all the primes from 0 to *N*, due to the number of trial divisions involved.

Is there a faster way?

## The Sieve of Erostothanes

There is, and it's one of the oldest ways of finding primes. It works like this:

1. Write down all the numbers *1 ... N*
2. Cross out every 2nd number after 2
3. Cross out every 3rd number after 3
4. Cross out every 4th number after 4
2. Continue until *sqrt(N)*

For instance, let's find all the primes <= 16. First, all the numbers:

     1  2  3  4
     5  6  7  8
     9 10 11 12
    13 14 15 16

Then, every 2nd number after 2... 

     1  2  3  _ 
     5  _  7  _
     9 __ 11 __
    13 __ 15 __

Then, every 3rd number after 3..

     1  2  3  _ 
     5  _  7  _
     _ __ 11 __
    13 __ __ __

and every 4th number after 4 (no change). Now we've got a tidy list of the primes <= 16: 1, 2, 3, 5, 7, 11, and 13.

This is not faster than trial division for any given number (you still need to do work for all *< sqrt(N)* beneath it) but it reduces dramatically the work needed over the range of numbers. You can see why intuitively: for instance, instead of performing an operation on every number *1 ... N* to see if it's divisible by 5, we perform an operation on every 5th number. 

In Go, we can do this pretty simply with a `bool` array of numbers. We'll assume everything is prime, and "cross it off" (mark it false) as we go.

```go
// performs the "sieve of Erostothanes" for a single number
func sieve(num int64, primes []bool) {
	for i := num * 2; i < int64(len(primes)); i += num {
		primes[i] = false
	}
}

// make an array and mark all numbers "prime"
num, _ := strconv.ParseFloat(os.Args[1], 10)
primes := make([]bool, int64(num))
for i := range primes {
    primes[i] = true
}

// mark all non-prime numbers
for i := 2; i <= int(math.Sqrt(num)); i++ {
    sieve(int64(i), primes)
}
```

# Testing Rotations

Once we've got our list of primes, we need to check each one to see if it's a *circular* prime. What's a fast way to generate each of its rotations?

One way would be to convert the number into a string, put its last character first, and then turn it back into a number, e.g. 

    1234 => "1234" => "4123" => 4123 

But consider the number of operations required to do this: if implemented naively, it's probably going to cost us at least one memory allocation (for the string), plus the time to parse/deparse the decimal representation. 

Computers are very fast at math, so let's see if we can do this entirely with math. For some number N:

1. Extract the last digit: *m = N mod 10* 
2. Multiply that digit by 10 if *N < 100*, 100 if *N < 1000*, etc:  *floor(log10(N)) ^ 10* 
3. Add the rest of the number, less the last digit: *m + floor(N / 10)*

Let's try it:

1. Extract the last digit: *1234 mod 10 = 4*
2. Multiply: *floor(log10(N)) ^ 10 = 4000* 
3. Add the rest of the number: *4000 + floor(1234/10) = 4123*

Okay, now we can wrap it up a nice Go function:

```go
func rotate(i float64) float64 {
	return math.Mod(i, 10)*(math.Pow(10, math.Floor(math.Log10(i)))) + math.Floor(i/10)
}
```

# Putting it Together

Now all we need to do is:

1. Loop through each of the primes
2. Check its rotations (one per digit)

```go
for i := range primes {
	if i > 1 && primes[i] {
		// this prime might be circular
		candidate := float64(i)
		ok := true

		// check all its rotations (1 rotation for 2-digit numbers,
		// 2 rotations for 3-digit numbers, etc.)
		var rotations = int(math.Floor(math.Log10(float64(i))))
		var primerotations = 0
		for j := 0; j < rotations; j++ {
			candidate = rotate(candidate)
			if (candidate < num) && !primes[int(candidate)] {
				// this rotation is not prime
				ok = false
				break
			} else if candidate < num && candidate != float64(i) {
				// this rotation is prime, so mark the prime as visited
				primes[int(candidate)] = false
				primerotations++
			}
		}

		// if all rotation were prime, output the number
		if ok {
			// count the number and all its rotations as circular primes
			circularprimes += (1 + primerotations)
		}
	}
}
```

Let's try it (you can find the full source code [here](https://github.com/jmcphers/circularprimes)):

    $ ./circularprimes 1000000
    2
    3
    5
    7
    11
    13, 31
    17, 71
    37, 73
    79, 97
    113, 311, 131
    197, 719, 971
    199, 919, 991
    337, 733, 373
    1193, 3119, 9311, 1931
    3779, 9377, 7937, 7793
    11939, 91193, 39119, 93911, 19391
    19937, 71993, 37199, 93719, 99371
    193939, 919393, 391939, 939193, 393919, 939391
    199933, 319993, 331999, 933199, 993319, 999331
    found 55 primes

And how'd we do on performance?

    real    0m0.055s
    user    0m0.050s
    sys     0m0.003s

That's 1/20th of a second to find all the primes under 1 million *and* test each for circularity. Not too bad.

# Final notes

This is a good example of how low-overhead Go is, but it doesn't really use any of Go's unique features in its implementation. If you wanted to do that, one useful trick would be to take advantage of the fact that the Sieve of Erostothane is extremely easy to parallelize, and Go has good concurrency primitives.

Here's a [concurrent prime sieve](https://golang.org/doc/play/sieve.go) written in Go for reference.

