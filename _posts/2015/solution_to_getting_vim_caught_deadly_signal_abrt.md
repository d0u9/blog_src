---
title: >-
  Solution to Getting "Vim: Caught deadly signal ABRT" after install python via
  homebrew
categories:
  - Science & Technology
  - Tools
tags:
  - Vim
  - Vim Plugin
  - Tools
  - Solution
abbrlink: 3956846196
date: 2015-11-23 18:54:00
---

Due to the new *[System Integrity Protection (SIP)](https://developer.apple.com/library/tvos/documentation/Security/Conceptual/System_Integrity_Protection_Guide/System_Integrity_Protection_Guide.pdf)* mechanism which is introduced with OSX 10.11 El captain, `pip` is nearly scrapped unless disable the SIP. However, if you are a python 3.X supporter who never use python 2.X again, you can simply install pyhon 3.X via [Homebrew](http://brew.sh/) and skip this post.

<!-- more -->

But for most people, especially for who uses [YCM](https://github.com/Valloric/YouCompleteMe#mac-os-x-installation), python 2.X is a necessity.

If you want to use python 2.X which is installed via homebrew, and want it to cooperate with YCM. You may encounter such a problem that vim could suddenly exit with an error message -- 'Vim: Caught deadly signal ABRT'.

This problem is discuss [HERE](https://github.com/Valloric/YouCompleteMe/issues/8), and I found the successful solution which resolved my problem.

1. Uninstall vim via `brew uninstall vim` command, this is a compulsory step. You have to unintall vim before the successive steps.
2. If you have install python 2.X via homebrew, unintsall it!
3. Install python: `brew install python --framework`.
4. Install Vim: `brew install vim`.
5. Reinstall your Vim plugins via vundle.
6. Re-compile your YCM: `python ./install.py --clang-completer`.
7. Enjoy it!

---

### ¶ The end
