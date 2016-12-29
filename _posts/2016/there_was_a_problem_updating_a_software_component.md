---
title: 'There was a problem updating a software component. Try again later...'
categories:
  - Science & Technology
  - Linux
tags:
  - Vmware
  - Linux
abbrlink: 1659865178
date: 2016-12-30 04:09:00
---

On Arch linux, after VMware Workstation have installed, you may encounter the error below when you booting the guest OS:

> VMware Tools - There was a problem updating a software component. Try again later and if the problem persists, contact VMware Support.

Before this error dumps, an update of VMware tool notification pops up which requests your permission. If you permit the update, during the downloading process(maybe after the downloading), the error will be reported.

<!-- more -->

This problem only appears on my Arch linux. Installing VMware Workstation on Debian is pretty smooth and functional.

After some googlings, I find the correct answer [here](http://askubuntu.com/questions/320713/vmware-tools-there-was-a-problem-updating-a-software-component-try-again-late). The question is asked on Jul 16'2013, however, the functional answer is posted on Nov 15'2016...

## How to solve

This error is largely due to the version mis-match of ncurses lib. VMware needs ncurses 5, however, the ncurses lib installed from pacman is 6.

There are two ways to settle this problem:

1. According to the answer posted on Askubuntu, one can simply make a symbolic
link from ncurses lib 6 to ncurses lib 5.

   ```bash
   cd /usr/lib
   ln -s libncursesw.so.6 libncursesw.so.5
   ```

2. Install **ncurses5-compat-libs** from [AUR](https://aur.archlinux.org/packages/ncurses5-compat-libs).

---

### ¶ The end
