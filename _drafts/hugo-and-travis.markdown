---
layout: post
title:  "Deploying Hugo with Travis CI"
date:   2016-11-09
categories: hugo, web development
---

As a part-time web developer, I used to build sites with Wordpress, but have recently become enamored with static site generators. There's just something ineffably elegant about having the site pre-built. There's no code on the web server to break, or patch, or hack. There's no server configuration to screw up. Static sites just Exist, in pristine immutability. 

Even better, a static site is usually constructed with plain text files, which means you can use version control systems like Git to manage your content. Want to experiment with an alternate version of the site without messing up the copy everyone sees? Or be able to roll back to any point in time? Or have a transparent log showing exactly who changed what and when? All of these capabilities and more, often implemented with varying degrees of success by big content management systems, are supported with simplicity and elegance by version control systems.

One big drawback to static sites, though, is that in order to *change* them, you have to *rebuild them*. This brings about the dreary requirement of having to *install and run software on a computer* in order to update your site. Even if it's not particularly needy software, it's a huge hurdle, especially compared to the convenience of updating content through a web interface offered by Wordpress and friends.

This will not do. 

So I set out with the goal of making it possible to have a site which uses version control, and static generator, **and** is easy to update from a website (just like Wordpress), without installing any software. I also had the goal of using only freely available, open-source software and tools. Here's how it works.

## The players

There are four major pieces to the system:

### The static generator

I've been very pleased with the [Hugo](https://gohugo.io/) site generator. It is robust, it's mature, it's fast, and -- perhaps most important of all -- it is virtually free of external dependencies and runs one every major platform without any fuss. If you've ever had to use a generator like Jekyll that requires you to wrangle with Ruby before you can get your site built, you know what a big deal this is.

### The code host

The code for the site lives on [Github](https://www.github.com/). Github gives you quite a lot of space for free if you're developing open source software. And as you're making a static site, you can probably use an open source license as you are by definition not building a site with secret bits. (Check with your lawyers first, of course.)

### The web host

In my case I'm hosting on [Dreamhost](https://www.dreamhost.com/), but any place that can serve HTML will work.

### The site builder

We need one more piece: Hugo has to run *somewhere*. It's going to run on [Travis](https://www.travis-ci.org/), a continuous integration system provided free of charge for small open-source projects.

## The big picture

Here's how things will work from a high level:

1. All the code for the site lives on Github.
1. A special file on Github includes instructions for building the site with Hugo.
1. When a new commit is made to Github, Travis CI will fire up and execute the instructions.
1. If Travis builds the site successfully, it deploys the built site to production.

## Making it work

### Step 1: Create your site

I'm not going to cover getting started with Hugo. Just read the [Hugo Quickstart Guide](https://gohugo.io/overview/quickstart/).

