---
title: 'Install the newest version of zsh shell in Linux'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Shell
  - Zsh
abbrlink: 3669365634
date: 2016-03-17 10:53:00
---

Due to the version of zsh in the Ubuntu 12.04's apt repository is too old to correctly source Zephyr's environment variables. I decided to install the newest zsh from source manually. After compiling and installation, when I changed my default login shell to my new zsh, however, some strange things happened.

The system told me that _you may not change the shell for xxx_ where xxx is my username. I was astonished by this message! This is my computer, why you can prevent me from changing my login shell? Well, you are my boss, I surrender....

<!-- more -->

After a lot of googling, finally, I got [this](http://askubuntu.com/a/479366/424680). Briefly, it is due to that my new zsh is not included in the `/etc/shells` file. After appending my new zsh in this file, problems are gone.

Thank you for your visiting....

---

### ¶ The end
