---
title: 'cmake简介(PART I: 基本使用)'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Cmake
abbrlink: 858347849
date: 2016-05-16 14:46:00
---

摘自维基百科：

> CMake是个开源的跨平台自动化建构系统，它用配置文件控制建构过程（build process）的方式和Unix的Make相似，只是CMake的配置文件取名为CMakeLists.txt。Cmake并不直接建构出最终的软件，而是产生标准的建构档（如Unix的Makefile或Windows Visual C++的projects/workspaces），然后再依一般的建构方式使用。

> “CMake”这个名字是"cross platform make"的缩写。虽然名字中含有"make"，但是CMake和Unix上常见的“make”系统是分开的，而且更为高级。

最近尝试了**autoconf**，**automake**，**libtool**等自动化构建工具，使用这些工具相比以前的手写Makefile来构建项目要简单了许多。但是，它们依然过于繁琐，而且对于跨平台来说不够友好。

之后突然想到在编译[youcompleteme](https://github.com/Valloric/YouCompleteMe)和[neovim](https://github.com/neovim/neovim)时用到的cmake命令，经过简单的学习发现cmake是一个很强大而且也比较简单的工项目构建工具。

这篇帖子是基于官方[tutorial](https://cmake.org/cmake-tutorial/)所写，扩充了官方文档中不太详细的部分。

<!-- more -->

# Cmake的安装

Cmake的安装非常的简单，官方已经提供了许多常用平台下的二进制文件发行版。

下载位置[https://cmake.org/download/](https://cmake.org/download/)。

这里以当前最新的cmake3.5.2版本为例，如果你和我一样使用的是Linux那么只需要下载*cmake-3.5.2-Linux-x86_64.tar.gz*或*cmake-3.5.2-Linux-i386.tar.gz*，解压到任何一个你觉得合适的目录下，将解压后的**bin**文件夹添加到`$PATH`环境变量中即可。

# Hello World 示例

创建一个**project**文件夹作为项目的根文件夹，并在其中创建一个**main.c**文件，内容如下：

```C
#include <stdio.h>

int main(int argc, char *argv[]) {
	printf("Hello World!\n");

	return 0;
}
```
# 使用Cmake来构建项目

## 创建CMakeLists.txt

Cmake使用**CMakeLists.txt**文件来进行配置。对于我们的Hello World文件，**CMakeLists.txt**极为简单：

```cmake
# 用来构建该项目所用cmake的最低版本。
cmake_minimum_required (VERSION 2.6)

# 项目名
project (Hello_World)

# 使用main.c来生成hello。
add_executable(hello main.c)
```

## 创建build文件夹

Cmake的一个有点就是它会将所有的中间文件，例如.o文件等放在一个文件夹中，从而不会让项目的源码变得混乱。在这里我们使用**build**文件夹来保存所有的中间文件。

```bash
mkdir build
cd build
```

## 开始构建项目

在**build**文件夹中执行:

```bash
cmake ..
```

这时会在**build**文件夹中生成一个Makefile文件（对于Linux用户来说，如果是其他用户可能会不同），下面只需要执行**make**命令就可以生成最终的二进制文件了。

```bash
make
```

执行该二进制文件可以看到：

```bash
./hello
Hello World!   # Output
```

这时的文件树结构如下：

```
.
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── cmake_install.cmake
│   ├── hello
│   └── Makefile
├── CMakeLists.txt
└── main.c
```

---

### ¶ The end
