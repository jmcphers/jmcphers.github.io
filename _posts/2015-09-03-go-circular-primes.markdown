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

> Cross out every 2nd number after 2
> Cross out every 3rd number after 3
> Cross out every 4th number after 4
> ...

This is not faster than trial division for any given number (you still need to do work for all *< sqrt(N)* beneath it) but it reduces dramatically the work needed over the range of numbers. You can see why intuitively: for instance, instead of performing an operation on every number *1 ... N* to see if it's divisible by 5, we perform an operation on every 5th number. 

In Go, we can do this easily.

{% highlight go %}
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
{% endhighlight %}

