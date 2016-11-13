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

    ssh-keygen -t rsa -b 2048 -N "" -f myproject -C "myproject@travis-ci.org"

The options are interpreted as follows:

+------------------+--------------------------------+
| `-t rsa -b 2048` | Create a 2048-byte RSA keypair |
| `-N ""`          | Do not protect the keypair with a passphrase. This is usually regarded as a bad idea, but it's unavoidable here since we'd have to give the passphrase to Travis CI anyway.  |
| `-f myproject`   | Save the private key as `myproject`, and the public key as `myproject.pub` |
| `-C myproject@travis-ci.org` | Add this comment to the key (helps to identify it) |
+------------------+--------------------------------+

You'll see some output that looks something like this: 

    Generating public/private rsa key pair.
    Your identification has been saved in myproject.
    Your public key has been saved in myproject.pub.
    The key fingerprint is:
    46:fd:17:b8:ba:09:7c:0f:fc:ad:80:91:0d:fe:8b:f5 myproject@travis-ci.org
    The key's randomart image is:
    +--[ RSA 2048]----+
    |                 |
    |         .   .   |
    |        o . . .  |
    |       o + . . . |
    |        S . o .  |
    |       o = . .   |
    |        + O      |
    |         = X .   |
    |        . + E..  |
    +-----------------+

**Copy the public key to your web host**

When you run the above command, it will make two files: a private key and a public key, which has the extension `.pub`. We're going to put the public key on your web host. Open the `.pub` file, and copy its contents to the clipboard. They will look something like this:

    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDqyEaRQzO0F1GKdPvs4q0lXtQ+YWewtIW1m...

Now, open a shell on your web host, as the user who has permissions to write files to the directory hosting the site. Edit the file `~/.ssh/authorized_keys` (create it if it doesn't exist), and paste the contents of the public key on the end of it. One easy way to do this from a Unix shell is to run this command:

    $ cat >> authorized_keys
    <press paste>
    <press Ctrl+D>

Of course, if you're an old hand at this you probably know that there are easier ways to do this depending on the hosts involved (such as [ssh-copy-id](https://linux.die.net/man/1/ssh-copy-id)).

Also, I'd be remiss if I didn't point out that you should *never* do this for a user who is highly privileged on your web host, as you want to limit the damage if the key somehow falls into the wrong hands. If needed, create a user who has only permissions to write the directory containing your web site. 

**Encrypt the private key for Travis**

Now, we definitely don't want that private key to ever touch Github, but we need Travis CI to use it, so the next thing we're going to do is encrypt it. 

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

Of course, you can edit the site locally if you like, but you can also edit it directly on Github. Just browse to any file in the site on Github and click the pencil icon in the upper right corner:

![Edit this file](/img/github-edit-file.png)

This will being up the file in a convenient little text editor. It's no substitute for a full-bore local development environment, but for making quick, drive-by updates for content, it'll do in a pinch, especially if you're not at your own computer.


