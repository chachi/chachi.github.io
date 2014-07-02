---
layout: post
title: ZeroMQ PUB SUB Compatibility
---

TL;DR - Only ever use a single version of ZMQ at once.

[ZeroMQ](http://zeromq.org) is an amazing foundation for building networked applications. Unfortunately, it can be light on the error messages just when you need them most. I was recently trying to communicate from a Raspberry Pi to my MacBook using ZMQ's PUB-SUB sockets and could not get my messages to be received by the MacBook. There weren't any error messages or warnings, so I was baffled.

Turned out that the Ubuntu 12.04 APT package version of [libzmq](https://launchpad.net/ubuntu/precise/amd64/libzmq-dev/2.1.9-1) was of the 2.* series, whereas the [Homebrew](http://brew.sh) package was of a more recent 4.* vintage. I found out from [this page](http://jaxenter.com/an-introduction-to-scriptable-sockets-with-zeromq-49167.html) that the PUB-SUB protocol changed in an incompatible and undocumented way between these versions, causing my messages to get silently discarded! I would hope the ZeroMQ developers take this sort of experience to heart and add some more logging mechanisms to help users uncover these sorts of silently discarded messages.

I fixed the problem by building a ZeroMQ 4.* version on the Pi and it worked flawlessly from there. Also, +1 for PyZMQ, which makes it stupid simple to communicate between applications with ZMQ.
