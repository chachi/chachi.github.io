---
layout: post
title: Homebrew - "The missing package manager for OS X"
---

Over the last six or so years, I've done development on both OS X and Linux, frequently switching between the two and occasionally cross-compiling from OSX targeting Linux. I got used to APT on Linux (Ubuntu/Debian), and always dreaded having to set up a new dev environment on OS X.

Sure, there were Fink and MacPorts, but I'm not sure they really made it much easier. They both had woefully out-of-date software and put your packages in offbeat places (/sw/ and /opt/local/, really?). They insisted on installing their own toolchains and I never even considered contributing back.

Fast-forward to 2013 and enter [Homebrew][homebrew]. Homebrew aims to make installing FOSS as easy and as natural as it is with Linux package managers.

Features I love:

- It uses /usr/local, so you don't have to modify your ```PATH``` or ```LD_LIBRARY_PATH``` or the rest.
- Your software is compiled with the default Xcode toolchain
- Homebrew avoids duplicating system resources.
- Packaging instructions (known as "Formula") are just short Ruby scripts.
- Want to build master instead of version _x_? Just use ```brew install --HEAD ...```
- Contributing back is just a matter of submitting a pull request on Github.

I use Homebrew exclusively on my Macs to get the FOSS that I need to get things done, and if Homebrew doesn't work just the way I need, I know I can just [contribute][pullrequest] back. No more special system paths, difficult setups, or outdated software.

[homebrew]: http://brew.sh "Homebrew - The missing package manager for OSX"
[pullrequest]: https://github.com/mxcl/homebrew/pull/22589
