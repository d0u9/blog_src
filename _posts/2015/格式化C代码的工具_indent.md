---
title: '格式化C代码的工具--indent'
categories:
  - Science & Technology
  - Tools
tags:
  - Linux
  - Tools
abbrlink: 3912087813
date: 2015-07-24 20:47:00
---

我们在写代码的时候总是会很注意自己的缩进，对齐等格式。好的代码风格不仅会大大的增加代码可读性，同时也会让其他阅读你代码的人感觉到你的代码中的艺术气息。

但是单纯的依靠我们写代码时注意，难免会产生失误，**GNU-indent**工具的出现解决了这一问题。

<!-- more -->

# For Mac OSX Users
如果你使用的是MacOSX，并且认真的看了开始的部分，那么你不难发现我们这里特地说到了**GNU-indent**。Mac OSX系统默认安装的**indent**是BSD的。因此本文中的参数可能不太适合您。为此我们首先要安装**GNU-indent**。

使用Homebrew安装GNU-indent:

```bash
brew install gnu-indent
```

Mac OSX中使用Homebrew安装的任何GNU工具默认都是带有`g`前缀的，因此我们需要使用`gindent`。

# 使用
我们这里仅仅按照Linux Kernel Coding Style来格式化我们的代码，使用如下指令：

```bash
indent -kr -i8 -ts8 -sob -l80 -ss -bs -psl <file>
```

笔者发现，使用上述参数会造成函数返回类型和函数参数表不在同一行，解决这个问题只需将最后的-psl参数去掉即可。

---

### ¶ The end
