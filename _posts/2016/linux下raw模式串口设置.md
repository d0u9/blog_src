---
title: 'Linux下RAW模式串口设置'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Serial Port
abbrlink: 37575732
date: 2016-12-20 09:10:00
---

目前新购买的计算机上，串口(uart，rs-232)已经难觅踪迹。但作为开发调试人员，串口依然广泛的被使用着，例如嵌入式开发，交换机配置等。

# RAW模式与非RAW模式

RAW模式简单的来说，就是发送端发动的二进制码原封不动的被接收端接收。

若干年前使用Windows下的**串口调试助手**对单片机串口进行调试就是使用的RAW模式，单片机发送的数据被原封不动的发送给PC端，PC端发送的数据也同样原封不动的发送回单片机。

非RAW模式下，系统会对串口收到的数据中某些具有特殊意义的字符或组合进行转义。这种工作模式的典型是在Linux下使用**minicom**配置交换机或串口登录其他Linux系统。

<!-- more -->

# 在Linux查看可用的串口

串口在Linux中被认为是一个普通的字符终端设备(tty)，因此串口的设备文件命一般为`/dev/ttyX`，其中的X依据不同的串口类型会为2个到若干个字符。

想要在Linux下查看可用的串口，一般的方法为使用`dmesg`命令来查看Linux的日志，从中找到串口的connect和disconnect信息，来确定具体的设备文件名。

在系统中执行如下命令：

```bash
dmesg | grep tty
```

在我的系统中（Debian 8），样例输出如下：

```
[    0.000000] console [tty0] enabled
[    0.552960] 00:01: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
[    3.019752] systemd[1]: Starting system-getty.slice.
[    3.019769] systemd[1]: Created slice system-getty.slice.
[    7.034310] usb 1-9: pl2303 converter now attached to ttyUSB0
```

可以看到我系统里有两个物理的串口，一个是`ttyS0`，另一个是`ttyUSB0`，其中`ttyS0`是我电脑主板上的原生串口，`ttyUSB0`是一根USB转串口线。

有时候电脑可能已经运行了很久，`dmesg`已经无法看到最初的信息，那么可以使用如下命令来查询:

```bash
ls -l /sys/class/tty
```

这条命令会列出电脑中所有的tty设备，包括虚拟机的tty设备。

我系统中的样例输出为：

```
lrwxrwxrwx 1 root root 0 Dec 20 13:18 console -> ../../devices/virtual/tty/console
lrwxrwxrwx 1 root root 0 Dec 20 13:18 ptmx -> ../../devices/virtual/tty/ptmx
lrwxrwxrwx 1 root root 0 Dec 20 13:18 tty -> ../../devices/virtual/tty/tty

Here, we ommit some virtual tty devices.

lrwxrwxrwx 1 root root 0 Dec 20 13:18 tty8 -> ../../devices/virtual/tty/tty8
lrwxrwxrwx 1 root root 0 Dec 20 13:18 tty9 -> ../../devices/virtual/tty/tty9
lrwxrwxrwx 1 root root 0 Dec 20 13:18 ttyS0 -> ../../devices/pnp0/00:01/tty/ttyS0
lrwxrwxrwx 1 root root 0 Dec 20 13:18 ttyS1 -> ../../devices/platform/serial8250/tty/ttyS1
lrwxrwxrwx 1 root root 0 Dec 20 13:18 ttyS2 -> ../../devices/platform/serial8250/tty/ttyS2
lrwxrwxrwx 1 root root 0 Dec 20 13:18 ttyS3 -> ../../devices/platform/serial8250/tty/ttyS3
lrwxrwxrwx 1 root root 0 Dec 20 13:18 ttyUSB0 -> ../../devices/pci0000:00/0000:00:14.0/usb1/1-9/1-9:1.0/ttyUSB0/tty/ttyUSB0
```

在最后我们能看到一些不是`virtual`的tty设备，其中一眼能分辨出来的是USB串口，此外还有四个tty设备，`ttyS0`，`ttyS1`, `ttyS2`和`ttyS3`，这四个设备我们需要逐个来试（抱歉，这里我也不知道怎么确定时哪一个，因为有些系统中platform下的串口也可能为实际的物理串口）。

# 设置串口为RAW模式

在Linux下系统的tty模式为非RAW模式，如果要调试单片机这种嵌入式设备，则需要将串口设置为RAW模式。

对tty的操作使用**stty**命令。

设置串口波特率到9600：

```bash
stty -F /dev/ttyX 9600
```

设置串口为RAW模式：

```bash
stty -F /dev/ttyX 115200
```

也可以在同一条命令中同时这是波特率和RAW模式：

```bash
stty -F /dev/ttyX 115200 raw
```

> 注意：我在Mac电脑上使用stty时，发现其命令行参数`-F`需要用`-f`来替换。

---

### ¶ The end
