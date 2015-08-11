---
title: '在Debian中禁用某一驱动的自动加载'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
  - Debian
abbrlink: 1604595148
date: 2015-08-11 08:31:00
---

最近在调试驱动的时候总是会遇到一件让人很恼火的事情，由于在编译内核的时候开启了input event debug这个选项，每次想在dmesg中查看调试打印的信息，但是映入眼帘的总是满满的evbug: Event ...之类的信息。虽然可以使用sudo rmmod evbug来手动的关闭，但是每次重新引导系统后该模块又会自动加载，很是烦人。经过了在Stackoverflow提问后，有大神给我指了一条明路。

<!-- more -->

# 具体操作步骤

1. 首先在`/etc/modprobe.d/`文件夹中创建一个和所要禁用模块通明的、以`.conf`位后缀的文件。例如我们这里要禁用`evbug`模块的自动加载，那么我们需要建立`/etc/modprobe.d/evbug.conf`文件。
2. 在上面创建的`evbug.conf`文件中添加`blacklist <modulename>`这一行，其中`modulename`为要禁用模块的名字。
3. 以root权限运行`depmod -ae`。
4. 使用`update-initramfs -u`重建initrd。

# 来源

Stackoverflow原帖：[http://stackoverflow.com/questions/30648383/how-to-disable-the-autoloading-of-a-specific-module-in-linux](http://stackoverflow.com/questions/30648383/how-to-disable-the-autoloading-of-a-specific-module-in-linux)

具体参考文献：[https://wiki.debian.org/KernelModuleBlacklisting](https://wiki.debian.org/KernelModuleBlacklisting)


---

### ¶ The end
