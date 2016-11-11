---
layout: post
title:  "Deploying Hugo with Travis CI"
date:   2016-11-09
categories: hugo, web development
---

As an occasional web developer, I used to build sites with Wordpress, but have recently become enamored with static site generators. There's just something ineffably elegant about having the site pre-built. There's no code on the web server to break, or patch, or hack. There's no server configuration to screw up. Static sites just Exist, in pristine immutability. 

Even better, a static site is usually constructed with plain text files, which means you can use version control systems like Git to manage your content. Want to experiment with an alternate version of the site without messing up the copy everyone sees? Or be able to roll back to any point in time? Or have a transparent log showing exactly who changed what and when? All of these capabilities and more, often implemented with varying degrees of success by big content management systems, are supported with simplicity and elegance by version control systems.

One big drawback to static sites, though, is that in order to *change* them, you have to *rebuild them*. This brings about the dreary requirement of having to *install and run software on a computer* in order to update your site. Even if it's not particularly needy software, it's a huge hurdle, especially compared to the convenience of updating content quickly, through a web interface, on any computer offered by Wordpress and friends.

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

I'm not going to cover getting started with Hugo. Just read the [Hugo Quickstart Guide](https://gohugo.io/overview/quickstart/), or do this in your Git repo. 

    hugo new site

Of course, very little of the rest of this will presume that you're using Hugo (any static generator will work).

### Step 2: Turn on Travis CI for your site

This is pretty simple -- just go to http://www.travis-ci.org/, make an account, click on your repositories, and flip the big switch next to the one with your site. 

### Step 3: Set up your deployment key

This is arguably the toughest part of the job. Since your Github repo is public, anyone can read it. Certainly we don't want any keys on there that could make it possible for strangers to deploy your site. 

**Create a new key pair**

First, run this command in your terminal:

    ssh-keygen -t rsa -b 2048 -C "yourproject@travis-ci.org"

This means "make me a new 2048-byte RSA keypair, and call it *yourproject@travis-ci.org*". 

**Copy the public key to your web host**

**Encrypt the private key for Travis**

### Step 4: Set up your Travis deployment script

Travis uses a special file called `.travis.yml` for build instructions. We need to tell it how to build your site. Because Travis uses a containerized build system, we'll have to give it instructions for building your site from scratch. 

The first thing we need to do is **decrypt the private key**, and set its permissions correctly.

    before_install:
      - openssl aes-256-cbc -K $encrypted_3d83f42c7c33_key -iv $encrypted_3d83f42c7c33_iv -in kingsgate5-travis.enc -out kingsgate5-travis -d
      - chmod 0600 kingsgate5-travis

Next we need to **install Hugo** (or, you know, whatever static generator you're using). You don't get sudo access on Travis, so we're going to just have to download and unzip the binaries.

    install:
      - wget https://github.com/spf13/hugo/releases/download/v0.17/hugo_0.17_Linux-64bit.tar.gz
      - tar xzvf hugo_0.17_Linux-64bit.tar.gz

Now that Hugo's installed, we can build the site. We do this with Hugo's `script` target:

    script: 
      - ./hugo_0.17_linux_amd64/hugo_0.17_linux_amd64 --theme=sunblind

Okay, great. Now Travis has the private deployment key, and a copy of the site, so all that's left is to `scp` the site onto the webserver.

    after_success:
      - cd public
      - scp -i ../kingsgate5-travis -o StrictHostKeyChecking=no -r * kingsgate5@kingsgate5.com:kingsgate5.com

Note that `.travis.yml` also supports a `deploy` target which has built-in capability for deploying to various hosts.

### Step 5: Profit

Once you've pushed your `.travis.yml` file, you're in business. 

