---
title: '在Ubuntu 14.04下编译YCM，cmake无法找到python3库文件'
categories:
  - Science & Technology
  - Tools
tags:
  - Linux
  - Vim
  - Vim Plugin
  - Tools
abbrlink: 4058215865
date: 2016-05-06 11:59:00
---

在[Compile YCM with a specific verion of python which is installed via pyenv](../../posts/1936695139.html)这篇文章中，我们试图自己编译YCM来使用指定的python版本，例如更新的python3，从而避免使用系统自带的老旧python。

但是当我自己在ubuntu 14.04下编译YCM时，总会遇到cmake无法正确找到python3库文件的错误，即使已经使用了`-DPYTHON_INCLUDE_DIR`和`-DPYTHON_LIBRARY`选项。

自己在YCM项目主页上提交了issue，但是并没有获得什么正确的回答，问题地址[https://github.com/Valloric/YouCompleteMe/issues/2113](https://github.com/Valloric/YouCompleteMe/issues/2113)。

<!-- more -->

废话少说：

要在ubuntu 14.04或某些老旧版本的linux中解决这个问题很简单，**安装最新的cmake**。是的`apt-get`安装的cmake太旧，所以会有这个问题，我猜测这是一个BUG。

---

### ¶ The end
