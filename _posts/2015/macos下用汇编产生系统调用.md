---
title: 'MacOS下用汇编产生系统调用'
categories:
  - Science & Technology
  - Programming Language
  - Assembly Language
tags:
  - MacOS
  - Assembly Language
abbrlink: 851584279
date: 2015-04-22 12:34:00
---

> 本文主要参考了该篇文章：[Making system calls from Assembly in Mac OS X](https://filippo.io/making-system-calls-from-assembly-in-mac-os-x/)

# 什么是系统调用

在（CSAPP-PARTII-Section8.1.2）中，讲到了系统调用的产生过程与用途。

陷阱是由于由执行某些指令专门产生的异常。陷阱的最重要的使用便是提供系统调用——在用户程序和系统内核之间的接口。

当用户需要从内核请求某些服务时，例如，读文件，创建进程，执行程序或终止目前的程序。为了能在有控制的情况下请求这些服务，处理器提供了一个指令叫做“syscall n”，这样用户在请求“服务n“时只需执行该指令即可。

<!-- more -->

# 如何用汇编产生系统调用

在32位程序中产生系统调用与在64位程序中产生系统调用是不同的。

下面我们就以经典的“Hello World”程序为例。分别用32位何64位实现。

## 原理分析

如果是使用C程序在MAC OSX下实现这个功能，我们应该使用的是`write()`，这里我们不使用`printf()`，我们需要直接与内核通信。

C程序代码如下：

```C
#include <unistd.h>
#define STR "Hello World\n"
 
int main(int argc, const char *argv[])
{
    write (1, STR, sizeof(STR));
    return 0;
}
```

现在我们将其改为汇编的实现。

首先我们需要知道其系统调用号，可以在[MAC OS 系统调用号](http://www.opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master)查询查到。

```
4 AUE_NULL ALL { user_ssize_t write(int fd, user_addr_t cbuf, user_size_t nbyte);
```

由此我们可以得知，`write()`函数的系统调用号为`4`。

## 32位程序的系统调用

32位系统产生系统调用的步骤如下：

1. 将函数需要传递的参数，按照从右到左的顺序，压入栈中。
2. 压入栈的数据需要是16字节对齐的。
3. 将系统调用号存入`%eax`中。
4. 产生`0x80`中断。

```Assembly
#as -arch i386 hello_world.s -o hello_world.o
#ld -macosx_version_min 10.9 hello_world.o -lSystem

.data
str:
    .ascii  "Hello World\n"

.text
.globl  _main
_main:
    pushl   $12                 #压入sizeof(str)
    pushl   $str                #压入str地址
    pushl   $1                  #压入STDOUT
    movl    $4, %eax            #写入系统调用号
    subl    $4, %esp            #16字节对齐(16-4*3) = 4
    int     $0x80               #调用中断
    addl     $16, %esp          #释放栈空间
 
    pushl   $0                  #压入exit()函数的值
    movl    $1, %eax            #写入系统调用号
    subl    $12, %esp           #16字节对齐（16-4）＝12
    int     $0x80               #调用中断
```

## 64位程序

64位程序相对来说要简单一点，但是与32位程序的方式确是完全不同的。OS X（还有GNU/Linux 和 任何除Windows除外的系统）在64位处理上都会遵循[System V AMD64 ABI reference](http://x86-64.org/documentation/abi.pdf)。在A.2.1可以找到有关系统调用约定的内容。

简单的总结如下：

1. 系统调用的参数保存在`%rdi`，`%rsi`，`%rdx`，`%r10`，`%r8`和`%r9`中。
2. 系统调用号放入`%rax`中。
3. 执行`syscall`指令。
4. 需要特别注意的地方是，MAC OSX系统中的系统调用号需要再加`0x20000000`。也就是说，如果我们要再MAC OSX中调用`write()`，则需要向`%rax`寄存器写入`0x2000004`而非`4`。

```Assembly
#as hello_world_64.s -arch x86_64 -o hello_world_64.o
#ld -arch x86_64 hello_world_64.o -macosx_version_min 10.9 -lSystem

.data
str:
    .ascii "Hello World\n"

.text
.globl  _main
_main:
    movq     $1, %rdi
    movq     str@GOTPCREL(%rip), %rsi     #此处与32位程序不同，64位程序采用PC－relative寻址方式
    movq     $12, %rdx
    movq     $0x2000004, %rax
    syscall

    movq     $0, %rdi
    movq     $0x2000001, %rax
    syscall
```

---

### ¶ The end

