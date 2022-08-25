---
layout: post
title:  "JavaScript: When an Array isn't an Array"
date:   2015-08-18 21:04:43
categories: javascript, bugs
---

Recently, I was investigating a JavaScript bug in which an array serialized differently when it was created in another window. Let's say `foo` is an array created in this window, and `bar` is an array created in another window.

    > var foo = ["a", "b", "c"]
    > var bar = window.opener.foo;

Now, let's try to serialize them. (`serialize` is not a real method, but it illustrates the problem.)

    > foo
    ["a", "b", "c"]

    > serialize(foo)
    "['a', 'b', 'c']"
    
    > bar
    ["a", "b", "c"]
    
    > serialize(bar)

    "{'0': 'a', '1': 'b', '2': 'c'}"

Wait, what? How could two ostensibly identical objects serialize differently? Here's another clue:

    > foo instanceof Array
    true
    > bar instanceof Array
    false

Now, how is it possible that `bar` isn't an array? It sure *looks* like an array, doesn't it? 

It turns out that the `Array` object in JavaScript is not a global construct but is *window-local*. So when you type this:

    > bar instanceof Array

You're actually asking not "is `bar` an array?" but "is `bar` an instance of **this window's** `Array` class?", to which the answer is `false`. 

Great. Thanks JavaScript. That's really straightforward.

So -- how exactly does one determine if an object is an array, regardless of where it comes from? 

The answer comes from none other than [Douglas Crockford](https://en.wikipedia.org/wiki/Douglas_Crockford), in his article [Remedial Javascript](http://javascript.crockford.com/remedial.html):

    Object.prototype.toString.call(value) == '[object Array]'

Yes, I'm not kidding. That is the only reliable way to see whether a JavaScript object is an array. Let's try it:

    > Object.prototype.toString.call(foo) == '[object Array]'
    true

    > Object.prototype.toString.call(bar) == '[object Array]'
    true

Newer browsers have `Array.isArray`, so there's that, and [this StackOverflow question](http://stackoverflow.com/questions/4775722/check-if-object-is-array) provides twenty different ugly and inelegant ways to determine whether an object is an array, which is a more eloquent argument than I could ever give for the complete brokenness of this particular feature of the language.
