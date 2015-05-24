---
title: 'GCC常用编译选项'
categories:
  - Science & Technology
  - Tools
tags:
  - Linux
  - GCC
abbrlink: 829865641
date: 2015-05-24 13:12:00
---

- -std

   指定gcc使用某一个版本的C标准进行编译。

   例：使用C99标准编译prog.c文件。

   `$ gcc -std=c99 prog.c`

<!-- more -->

- -masm

   让GCC产生Intel格式的汇编，而不是AT&T格式的汇编。
   例：

   `$ gcc -S -masm=intel prog.c`

- -march

   指定目标CPU架构以进行对应的优化。

   例：生成在早期X86架构CPU上可执行的代码（例如不使用跳转移动而使用跳转）。

   `$ gcc -arch=i686 prog.c`

- -m32

   在64位机上产生32位兼容的代码。

   例：

   `$ gcc -m32 prog.c`

- -c

   预编译，汇编，编译代码，但不进行链接。最后产生目标文件 .o。

   例：

   `$ gcc -c prog.c`

- -S

   生成汇编代码.s文件。

   例：

   `$ gcc -S prog.c`

- -static

   静态链接。即将库文件代码嵌入到最终的可执行文件中。有时某些库可能仅安装了动态链接文件.so，因此可能产生链接错误。这时需要安装对应的static库，例如 yum install glibc-static。

   例：将main.o和libc.a进行静态链接生成可执行文件。

   `$ gcc -static main.o ./libc.a`

- -fPIC

   生成position-independent的代码，可以供动态链接使用。

   例：见下方 -shared 选项。

- -shared

   生成.so动态链接库文件。

   例：将prog.c 与 lib.c 编译成.so的动态链接库文件。

   `$ gcc -shared -fPIC -o libvector.so prog.c lib.c`

- -fno-asynchronous-unwind-tables

   使用-S编译选项时，使生成的汇编文件中不包含带有cfi的directives.

---

### ¶ The end

