---
title: "Copying and Pasting Binaries with OpenSSL"
date: 2022-08-22
---

## Sometimes You Can't `scp`

We've all been there: you have terminals open on two different remote hosts, and need to copy a file between them. Unfortunately, despite the fact that they are **LITERALLY NEXT TO EACH OTHER** on your screen, they just can't see each other, because they're each behind so many layers of VPNs and bastion hosts and NATs and different accounts and time zones that a direct connection feels impossible. Or, at least, overwhelming.

## Magic Wormhole

My go-to solution to this is [Magic Wormhole](https://magic-wormhole.readthedocs.io/en/latest/). So easy!

Sender:

```
$ wormhole send foo.bin
Sending 124 byte file named 'foo.bin'
On the other computer, please run: wormhole receive
Wormhole code is: 8-plantain-chips
```

Receiver:

```
$ wormhole receive 8-plaintain-chips
```

It uses a 3rd-party service, though, and so both machines have to be connected to the Wild Internet, plus you have to install Magic Wormhole on them, and sometimes that's hard too; it uses Python, so then you start trying to install Python, but then you don't have admin rights, and you know how *that* ends.

## OpenSSL

For small-ish files, an elegant approach is to use OpenSSL and the clipboard of the system connected to both hosts. You'll find OpenSSL on almost every macOS and Linux base system, so you almost certainly won't have to install it.

Here's how:

### Base64 Encode on Source

Let's say we've got a file called `foo.bin` we'd like to transfer between hosts. First, we ask OpenSSL to encode the file using [Base64](https://en.wikipedia.org/wiki/Base64). Note that this does not *encrypt* the file's contents; it just turns them into plain text you can copy and paste. It'll look like a whole bunch of garbage. That's what it's supposed to look like.

```
$ openssl base64 -in foo.bin
dcJS3uOIv94OMk+turrEGTo0y5q+vZDcxX3ygN6BLzXrrj0eq9XMTK/97JculIWF
OR4wBuF1mFFjiIq73y443yvpWAw//7WQ3kbK6hha+EQ9PWxA3OVPkhTLl1V0kwJ0
/t1UrMpnGFK0OESxt6VhunTdWPQ751HSzWZyeCWse78GM8tx3e/KOY3PWfVpUtjc
HLCmZSSWfgduc7WestpepSiX/1XUwiAlZ+tHMzRG0TaWNPaXlJxEY6oowIVZcHfm
...
```

Now, copy that whole mess to your clipboard. Again, this only works for relatively small files; for bigger ones, you are liable to exhaust your system's clipboard capacity and/or your mousing hand.

### Base64 Decode on Target

Next, switch to your target host. Type as follows:

```
$ openssl base64 -out foo.bin -d << EOF
*paste here*
EOF
```

Press `Enter`, and *ta-da* -- the file will magically appear on the target, byte for byte identical with the original. You can `md5sum` them both if you're 


