---
layout: post
title:  "Password Management"
date:   2016-05-08 12:32:32
categories: security
---

## Introduction

Everyone has their own way of managing the dozens of passwords they need to sign on to online services. Some people use the same password everywhere. Others use sticky notes or a notepad shoved into a desk drawer. If you're technically inclined, you might use a password manager. I've been using the same password management system for almost half a decade now, and I've been very pleased with it. I'm publishing it here in hopes it will be helpful to others.

## For the Non-Technical

I fully admit that my system is a little on the paranoid side. For the average user who just wants to improve their security, here's what I usually recommend, in descending order of value:

1. If you do nothing else, make a very good e-mail password and never use it on any other account. Your e-mail account is almost as good as a skeleton key. It can be used to reset your password on almost all of your other accounts. That means that if you use your e-mail password on another site, and that site's passwords are stolen, whoever gets the password may also be able to hack into your e-mail account. [And that will be very, very bad](http://www.theatlantic.com/magazine/archive/2011/11/hacked/308673/?single_page=true).

1. If you already have a good e-mail password, turn own two-factor authentication for your e-mail account, to give it an extra layer of security. Two-factor authentication (sometimes abbreviated by well-meaning nerds as 2FA) just means you need something besides your password to log in. Usually that something is a code you see on your phone that changes every 30 seconds (making it useless even if captured by an attacker).

1. If you've already got a good e-mail password, change your passwords so they don't contain personal information. In addition to being easier to guess, these kinds of passwords have statistical characteristics that make them easier to crack. For instance, a year is considerably easier to crack than a random 4-digit number since most 4-digit numbers do not represent a recent year. Many passwords are even "pre-cracked" in [rainbow tables](https://en.wikipedia.org/wiki/Rainbow_table), requiring almost no effort at all for an attacker to defeat. 

1. If you're already avoiding personal information, go one step further and choose nonsense for your password. You don't even need a computer to make a good nonsense password; [Diceware](http://world.std.com/~reinhold/diceware.html) is one method which doesn't require you to use one at all. A password containing a few random words is actually much more secure than the shorter jumble of letters and numbers you've been taught is secure (there is a comic later in this post which explains the math).

1. If you already use strong passwords, and don't share them among sites, you probably forget them from time to time. And that's where a password management system comes in.

## Requirements

These are my own requirements for a password management system. Yours may be different.

1. The software components of the system must be *open source*, to ensure that the system does not contain hidden backdoors, and to guarantee the longevity of a system. Password management systems which rely on closed source or proprietary software are only as trustworthy and long-lived as the corporations which own them. Unfortunately, this rules out the most popular password management services, including [1Password](https://agilebits.com/) and [LastPass](https://lastpass.com/). (Many intelligent, security-minded people have no problem trusting these corporations with all their passwords. This post isn't a dig at them, I'm just a little more paranoid.)

1. The system must be able to survive a *catastrophic loss of equipment*; for instance, a fire which destroys everything in my home should not also render me unable to access my online accounts.

1. The system must be *extremely difficult to compromise*, even if an attacker has access to the encrypted data containing the passwords.

1. The system must be *accessible from multiple devices* and operating systems. For me that's a desktop, a laptop, and a mobile phone.

1. The system must not have a *single point of failure*. It should never be possible for the disclosure of a single piece of information, or the failure of a single piece of hardware, to render the system insecure or unusable. 

1. The system must be able to meet the *password requirements of many different services*: for instance, some require special characters while others do not allow them, others require that a password be changed every N months to a new password, etc. There are many stateless password management systems which are able to work without storing any passwords at all; they derive a password from some secret information combined in a known way with the site or service. Unfortunately these invariably are unable to produces passwords which meet the requirements of every service (which are often mutually exclusive), which means you have to remember how you modified the password to meet the service's requirements--a difficult task for rarely used services. Even if they were able to produce passwords which met requirements, they leave you unable to change your password if it expires or is compromised. 

## Implementation

My own system works as follows:

### Password Storage

The passwords themselves are stored in a [KeePass](http://keepass.info/) database. KeePass is not beautiful, and it does not have beautiful client software, but it's open source, and has several clients available for every platform.

The KeePass database is locked with a combination of two things:

1. A strong passphrase. 

1. A keyfile, which is a file containing a large number of random bytes. This is a somewhat unique feature to KeePass: you can use both a keyfile *and* a password to lock a database; in order to decrypt the passwords, you need to have both. It's something like two-factor authentication; that is, it's proof by something you know (the passphrase) and something you have (the file). 

Both of these things are required to unlock the database and access the passwords.

### Password Storage Storage

A typo? No. Once you've got your passwords stored somewhere, you need to to figure out where to put that storage. It needs to be accessible to you at all times, but it also needs to be as inaccessible to attackers as possible.  Fortunately, there are no shortage of cloud storage providers these days who are willing to give you a small amount of storage for free, and passwords don't take up very much space!  In recent years, authentication for cloud storage providers has gotten a lot better, too. You can get a free account protected by two-factor authentication at [Dropbox](http://www.dropbox.com/), [OneDrive](http://www.onedrive.com) (Microsoft), or [Google Drive](http://drive.google.com/). 

I put the password database itself (the KeePass `.kdbx` file) on one cloud storage provider and the keyfile on another cloud storage provider. Both cloud storage providers have login systems that require two-factor authentication.

If an attacker were to somehow hack into my cloud storage account and retrieve the password database, they would also need to hack into a *second* cloud storage account to get the keyfile, or vice versa. This is not impossible, but it's exponentially more difficult than hacking into a single service.

### Password Generation

If you think that a password with a jumble of letters and numbers and special characters is the most secure, then I have good news for you: a handful of words is actually more secure. As usual XKCD breaks down the math better than I could:

![Password Strength](/img/password_strength.png)

It's important to note that the words must be *random*. For instance, in actual usage the word "happy" is likely to be followed by words such as "birthday" or "new". If you base your passphrase on words from a real sentence, then your phrase will not be secure because of the statistical relationship between successive English words. 

There are four rules I follow when adding a password for a new service:

1. Unless there is good reason not to, sign into the service using a unique password rather than using "sign in with Facebook" or "sign in with Google". Connecting your identity across services can reduce privacy, and if you're not careful it can also give services the ability to perform unwanted actions on your behalf elsewhere (posting to Facebook, etc.)

1. If the service will only be used rarely, generate a long, random password. This will be cut-and-pasted from the password manager when needed. 

1. If the service will be used frequently, take the extra time to generate a unique, memorable passphrase. If you type in a passphrase frequently, you will quickly remember it and won't need your password manager for that service. I use an Android app based on [Diceware](http://world.std.com/~reinhold/diceware.html) to generate memorable passphrases of the `correct horse battery staple` variety. 

1. If the service supports two-factor authentication, use it! Two-factor authentication provides a huge increase in security -- possibly more than all the other advice in this post combined. [This is why you should use it.](https://blog.codinghorror.com/make-your-email-hacker-proof/)

## Considerations

### Devices

The largest weak point of the system is that the devices that I use every day (including my laptop and mobile phone) contain everything that's needed to unlock the database: the KeePass database, the keyfile, and a keyboard on which I regularly enter my passphrase. If an attacker were able to access the contents of the storage on these devices and install a keylogger of some kind, they could easily get access to all my passwords. 

The typical mitigation for this is to use a hardware device such as a [YubiKey](https://www.yubico.com/) to store the master password, or an ordinary USB stick to store a keyfile. However, I'm hesitant to take this approach, as it violates a number of my requirements. First of all, it introduces a single point of failure, and secondly, it prevents password access on my mobile phone, which is important to me since it is often the only computing device I have.

### Storage

It's somewhat risky to store all of the password data in the cloud, since a corporation has access to it. However, the strong encryption on the data makes it very unlikely that the corporation would be able to do anything useful with the data even if they had the desire to. 

Another approach might be to use a fully peer-based protocol such as [BTSync](https://www.getsync.com/) to store the password database and/or the keyfile. One advantage of this system is that the data would never be stored on a third-party system. An even bigger advantage is that applications which interface with my password database could be denied network access, which provides a stronger guarantee that they will not send any information where it shouldn't go.

However, the disadvantage is that the burden is on me to ensure that there's an off-site system to which the data is being backed up regularly, and to ensure that backups are available in the case of database corruption. 



