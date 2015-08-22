---
title: 'Linux驱动开发--ioctl入门'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linux Driver
abbrlink: 2768311140
date: 2015-08-22 15:39:00
---

在Linux系统中，`ioctl()`系统调用可以说是一个万金油，任何无法用标准文件操作完成的功能，例如修改网卡物理地址，修改串口波特率等操作均可以使用`ioctl()`系统调用来完成。

`ioctl()`系统调用的本质是自己创建了一套子命令系统，用以对某一个设备的特有属性进行控制。

<!-- more -->

# 用到的函数或宏定义

## ioctl()函数

`ioctl()`函数和`read()`，`write()`函数一样，存在于`struct file_operations`结构体中。这也就是说我们需要针对自己的设备来实现特有的`ioctl()`。

这里需要特别特别注意的是，如果你在阅读**Linux Device Driver 3rd ed**的话，书里使用的内核版本是2.6.xxx（具体记不清楚了）。在这个版本的内核中，`struct file_operations`结构体中的确包含的是`ioctl()`方法，原型如下：

```C
int ioctl(struct inode *, struct file *, unsigned int, unsigned long);
```

但是，由于`ioctl()`会使用到BKL（Big Kernel Lock）,再由于之后打算将BKL剔除出内核代码导致`ioctl()`也被相应的替换成了`unlocked_ioctl()`，该函数的原型如下：

```C
/**
 * @filp: indicates the file on which operations will be taken.
 * @cmd: sub-commands indicates which actions will be taken.
 * @arg: arguments which are passed in.
 */
long (*unlocked_ioctl) (struct file * filp, unsigned int cmd, unsigned long arg);
```

该函数的返回值最终会作为用户空间`ioctl()`函数的返回值。函数中的第三个参数`arg`不仅可以用来传值，也可用来传递某个空间地址。

## 与用户空间交换数据

在与用户空间交换数据的时候，不仅可以用`copy_from_user()`和`copy_to_user()`，还可以使用一些专门设计用来传递小数据的函数，如下：

```C
put_user(datum, ptr)
__put_user(datum, ptr)

get_user(local, ptr)
__get_user(local, ptr)
```

这些函数定义在`<asm/uaccess.h>`中，由于`<linux/uaccess.h>`也包含了`<asm/uaccess.h>`，因此可以直接包含`<linux/uaccess.h>`。

## ioctl编号

除此之外，在此处我们还要用到一些辅助的宏，用来定义我们自己的`ioctl()`号。

对于每个`ioctl()`会有唯一的一个编号使得该调用和某个驱动匹配。分配`ioctl()`的编号并非随机，而是需要使用`_IO(type, nr)`，`_IOW(type, nr, datatype)`，`_IOR(type, nr, datatype)`，`_IOWR(type, nr, datatype)`这几个辅助宏。

详细的讲解可以参见这里：[http://www.makelinux.net/ldd3/chp-6-sect-1](http://www.makelinux.net/ldd3/chp-6-sect-1)。

## 预先定义的命令

虽然ioctl主要被驱动使用，但还是有一部分会被内核所识别。具体的也可以参见：[http://www.makelinux.net/ldd3/chp-6-sect-1](http://www.makelinux.net/ldd3/chp-6-sect-1)。

# 实现我们自己的ioctl系统调用

在此，我们要实现一个简单的例子，这个例子会在**/dev/**目录中创建一个叫做**ioctl-test**的设备文件。在默认情况下，例如驱动刚加载后，读取该文件可以返回10条**Hello World!**字符串。我们可以通过自己定义的ioctl接口来修改期望的返回条数，和返回内容。

# 用户空间代码

首先我们看下用户空间的代码（随便写的，请忽略代码风格）：

```C
#include <stdio.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>

#define NEW_STR     "Hello, Linux!!\n"
#define NEW_STR_LEN sizeof(NEW_STR)

#define IOCTL_IOC_MAGIC 'd'

#define IOCTL_RESET _IO(IOCTL_IOC_MAGIC, 0)
#define IOCTL_LOOPNR    _IOWR(IOCTL_IOC_MAGIC, 1, int)
#define IOCTL_STR   _IOW(IOCTL_IOC_MAGIC, 2, int)

struct ioctl_str {
    int str_len;
    char *str;
};

int main(int argc, char *argv[])
{
    char *str = (char *)malloc(NEW_STR_LEN);
    memset(str, 0, NEW_STR_LEN);
    memcpy(str, NEW_STR, NEW_STR_LEN);

    struct ioctl_str str_io = {
        .str_len = NEW_STR_LEN,
        .str = str,
    };

    int fd = open("/dev/ioctl-test", O_RDONLY);
    /*int err = ioctl(fd, IOCTL_RESET); //reset*/
    /*int err = ioctl(fd, IOCTL_LOOPNR, 5);*/
    int err = ioctl(fd, IOCTL_STR, &str_io);
    printf("retval = %d\n", err);

    return 0;   
}
```

我们对该设备定义了3个命令，分别为`IOCTL_RESET`，`IOCTL_LOOPNR`和`IOCTL_STR`。为这三条命令传入响应的参数，可以分别将设备状态重置，修改设备返回字符串的条数和修改返回的字符串内容。

第35行 至 第37行包括注释掉的内容为测试样例。

第35行的代码会将设备恢复至初始状态，该命令不需要参数。

第36行的代码会修改返回字符串的次数为5次，其传入的参数方式为值传递。返回值为修改前的打印字符串次数。

第37行的代码会修改返回的字符串内容，其传入参数的方式为地址传递。此处，返回的字符串会被修改为**NEW_STR**，即**Hello, Linux!!\n**。

## 驱动模块的编写

为了简单起见，我们这里不详细描述设备文件的`read()`方法。如果不明白请查阅资料或者看看我之前的博文。

与*LDD3*相比，新的内核使用`unlocked_ioctl()`来取代了老的`ioctl()`用以摆脱BKL。因此我们需要将设备文件的`struct file_operations`结构中的`unlocked_ioctl`字段指向我们自己的`ioctl()`，我在这里将改函数取名为`ioctl_ioctl()`。

在`ioctl()`中我们要完成如下操作：

1. 首先我们需要检查参数是否正确，这包括检查魔数是否相符和子命令的编号范围是否正确。
2. 根据读取防线对参数地址访问进行验证，这需要使用access_ok()函数。
3. 根据传入的命令号分情况处理。
4. 读取用户空间的数据。

### 调用号定义

在此，我们使用魔数`d`作为我们ioctl调用的类型码。

定义的代码如下：

```C
#define IOCTL_IOC_MAGIC 'd'

#define IOCTL_RESET _IO(IOCTL_IOC_MAGIC, 0)
#define IOCTL_LOOPNR    _IOWR(IOCTL_IOC_MAGIC, 1, int)
#define IOCTL_STR   _IOW(IOCTL_IOC_MAGIC, 2, int)
```

### 用户空间地址检查

使用`access_ok()`函数可以对传入的用户空间地址进行检查。`access_ok()`函数的原型如下：

```C
/**
 * @type: 只能取VERIFY_READ或VERIFY_WRITE。如果既要读取又要写入，则为
 * VERIFY_WRITE，因为它是VERIFY_READ的超集。
 * @addr: 用户空间传参的地址。
 * @size: 用户空间传参数据的大小。
 */
int access_ok(int type, const void *addr, unsigned long size);
```

需要注意的是，从`ioctl()`的角度来看，读操作和写操作是相对于用户空间的代码来说的。也就是说，写操作是用户往文件写，读是用户从文件读。而对于`access_ok()`来说，读和写是相对于内核来讲的，读是从用户空间读取数据到内核空间，而写操作是从内核空间向用户空间写。

**此外需要注意的是：`access_ok()`这个函数对用户空间地址的检查时粗略的，且和所影星的硬件架构有关。在X86架构上，该函数仅仅判断了我们需要操作的地址是在整个物理地址空间的低位（因为内核代码处于高位）。**

> 参考：[http://stackoverflow.com/questions/30857584/can-i-pass-an-integer-to-access-ok-as-its-second-argument](http://stackoverflow.com/questions/30857584/can-i-pass-an-integer-to-access-ok-as-its-second-argument)

### 操作权限检查

此外，在处理具体的命令号时，我们还要判断是否有权限。在例子中，我们修改打印条数的操作需要超级权限。在内核中使用如下的代码片段来判断是否具有合适的权限：

```C
if (!capable(CAP_SYS_ADMIN))
    return -EPERM;
```

完整的操作权限定义在`<inux/capability.h>`中。其中我们可能会用到的权限如下：


```C
CAP_DAC_OVERRIDE
    The ability to override access restrictions (data access control, or DAC) on files and directories.

CAP_NET_ADMIN
    The ability to perform network administration tasks, including those that affect network interfaces.

CAP_SYS_MODULE
    The ability to load or remove kernel modules.

CAP_SYS_RAWIO
    The ability to perform "raw" I/O operations. Examples include accessing device ports or communicating directly with USB devices.

CAP_SYS_ADMIN
    A catch-all capability that provides access to many system administration operations.

CAP_SYS_TTY_CONFIG
    The ability to perform tty configuration tasks.
```

### ioctl的具体实现

最后我们只需使用`switch case`语句来根据命令跳转到对应的分支执行就好了。

对于字符串的传输，我们可以将内容写入到某个用户空间地址，然后再在`ioctl()`中使用`copy_from_user()`将内容拷贝到内核即可。

# 完整的代码

在这里找到：[https://github.com/d0u9/Linux-Device-Driver/tree/master/07_ioctl](https://github.com/d0u9/Linux-Device-Driver/tree/master/07_ioctl)

---

### ¶ The end
