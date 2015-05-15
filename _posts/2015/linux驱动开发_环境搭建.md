---
title: 'Linux驱动开发--环境搭建'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
abbrlink: 569064857
date: 2015-05-15 08:30:00
---

# 介绍

本文使用的Linux发行版为：Debian 7 wheezy。

其实驱动开发并不需要什么特殊的环境，但是为了更好的调试，我们需要对内核重新编译，以打开某些调试选项，让我们在遇到错误时可以窥探到内核的内部。

这里主要介绍如何在Debian 7 wheezy中如何打开部分调试选项并且重新编译、加载内核。

<!-- more -->

# 需要准备的工具

1. 安装了Debian 7 wheezy的计算机一台，最好时X86架构的，本人不知道在其他的架构上是否存在差异。
2. 将这台计算机可靠的接入互联网，并能够连接到源服务器。

# 开始

## 下载内核

我们可以从官方下载内核，官方的下载地址为[https://www.kernel.org/](https://www.kernel.org/)。更为简便的方法是使用debian的`apt-get`工具。

执行以下命令获得内核源码：

```bash
apt-get update && apt-get install linux-source-3.2
```

需要注意的是，当你看到这篇文章的时候，更新源中的内核版本可能已经不是3.2。那么你需要自己确定下当前更新源中包含哪些版本的源码。

除此之外还需要注意的是，如果你是从官方下载内核，请慎重选择内核版本。例如当前的wheezy使用的内核版本为3.2.XX，如果你下载并安装了最新的3.18内核，可能会造成一些奇怪的问题发生，这主要是因为系统的很多模块是与内核版本有关的。因此最好不要使用最新版本的内核，同时也最好不要使用非稳定版的内核。

## 解压并clean内核源码

使用apt-get下载的内核存放在`/usr/src/`下面。我们进入该目录并使用`tar`命令来解压源码。

```bash
tar -xvf linux-source-3.2.tar.bz2
```

解压后，你可以将改文件夹命名成一个方便自己记忆的名字，这里假设为：linux-3.2.65-development。

进入该文件夹后执行如下命令删除曾经有过的配置文件（应该是没有的，但是为了保险起见，还是执行下好）。

```bash
make mrproper
```

执行如下命令clean一下

```bash
make clean
```

## 配置内核

下面需要进行内核的配置，为了防止我们没有打开debian自身需要的特性，可以将当前内核的配置文件复制过来。

```bash
cp /boot/initrd.img-3.2.0-4-amd64 ./.config
```

之后，在此基础上我们再开启我们需要的编译选项。

进入内核配置界面执行如下命令（可能会提示却少某个库，使用apt-get安装上就好咯）

```bash
make menuconfig
```

在Kernel hacking选项中有大量的调试工具开关，我们可以按照自己的需要勾选。我是按照《Linux Device Driver》这本书中第四章配置的。

除此之外，如果你想让新编译的内核和当前内核共存的话，请在`General setup -> Local version`中填写一个后缀，例如`-development`，用以区分模块存放目录。

## 编译内核

在内核配置完成后，我们就可以开始编译了。首先我们需要编译内核镜像，执行下面命令：
```bash
make bzImage
```

内核编译完成后，我们需要编译内核模块。执行如下命令：

```bash
make modules
```

## 安装

我们首先要安装的是内核模块，这样才能在系统引导的时候，内核找到需要的相应的组建。

```bash
make modules_install
```

之后将内核镜像放进`/boot`目录中，执行如下命令：

```bash
cp arch/x86_64/boot/bzImage /boot/vmlinuz-3.2.65-development
```

由于系统在引导的时候需要提前加载某系模块，因此我们要将内核模块生成一个压缩归档文件供系统引导时挂载使用。

```bash
mkinitramfs /lib/modules/3.2.65-development -o /boot/initrd.img-3.2.65-development
```

其中的`/lib/modules/3.2.65-development`是在执行了`# make modules_install`之后生成的，因此你的名字和我这里的名字可能不一致。

向grub中添加新内核的引导。这个很简单，只需要执行如下的命令即可：

```
grub-mkconfig > /boot/grub/grub.cfg
```

## 重启&享用

重新引导你的计算机，在grub界面你就能看到新添加的内核咯。快乐的使用它吧。

---

### ¶ The end

