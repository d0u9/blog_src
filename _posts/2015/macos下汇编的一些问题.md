---
title: 'MacOS下汇编的一些问题'
categories:
  - Science & Technology
  - Programming Language
  - Assembly Language
tags:
  - MacOS
  - Assembly Language
abbrlink: 506267438
date: 2015-04-19 15:41:00
---

最近在MacOS下写汇编的时候遇到了一些问题，现在总结如下：

# 验证环境说明

- 硬件：macbook pro i5 处理器

- 软件：MacOS 10.9

<!-- more -->

# 开发环境搭建

## 汇编器**as**

我使用MacOS自带的**as**汇编器。当然，如果你喜欢使用intel格式的汇编代码，可以通过homebrew安装**nasm**。

安装：在安装了X-Code Command-Line-Tools之后就默认安装了**as**工具。

## 链接器**ld**

我使用MacOS自带的**ld**链接器。

安装：在安装了X-Code Command-Line-Tools之后就默认安装了**ld**工具。

## 反汇编工具**objdump**

使用brew安装binutils工具包即可。

```bash
brew install binutils
brew install binutils
```

# MacOS汇编与Linux GCC汇编格式的不同

这里仅仅是入门时一些常用格式的不同点，并非所有的不同点。不过在后续的学习中，新发现的不同点我会补充。

## 平台声明的不同

在MacOS下，要声明所写的代码是32位的，需要在汇编文件的顶端加入如下的声明。

```Assembly
.code32
```

同时编译的时候需要在对as加入参数`-arch i386`。

## 段声明的不同

在MacOS下，不能使用~~`.section .data`~~应该直接使用:

```Assembly
.data
.text

MOV $1, $2
```

## 程序的入口点定义不同

在MacOS下的汇编中，我们不使用`_start`来标记程序的开始点，而是使用:

```Assembly
_main
```
来声明程序的开始点。

## 系统调用不同

MAC OS X中没有`.int`，需要用`.long`来代替。

## 其他

到目前为止（07-08 2014）我暂时只发现了这四点。不过克服这四点，我们已经能够写出《Professional Assembly Language》第四章中的例子程序了（该程序在英文版77页）

---

### ¶ The end

