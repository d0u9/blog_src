---
title: '[译]具有魔法的宏--container_of()'
categories:
  - Science & Technology
  - Programming Language
  - C
tags:
  - Linux
  - Linux Kernel
  - C
abbrlink: 186012312
date: 2015-07-16 16:30:00
---

> 转自http://broken.build/2012/11/10/magical-container_of-macro/

> 当你开始学习内核，开始阅读内核源码时，早晚你都会遇到这个具有魔力般的宏定义。

# 引言

这个宏能用来做什么？准确的说，这个宏能名副其实。它接受三个参数－－一个指针，一个container的类型名，还有一个指针所代表的成员名。这个宏最后会返回一个新的，指向包含这个指针的container的地址。这个宏的确非常的有趣，但是它具体事如何工作的呢，本文将向您介绍。

<!-- more -->

第一幅图帮助那些没有理解上面内容的人了解`container_of(ptr, type, member)`宏的原理。

![](http://oyui6c341.bkt.clouddn.com/images/2015/具有魔法的宏_container_of/01.png)

下面是具体的宏定义代码（该代码定义在/include/linux/kernel.h)

```C
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:    the pointer to the member.
 * @type:   the type of the container struct this is embedded in.
 * @member: the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({          \
    const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```

第一眼会觉得这个宏很难理解，而且事实上也是这样的，下面我们一步一步的理解。

# 表达式中的语句

吸引你的低一点应该是整个表达式的结构了吧。我们希望整个语句返回一个指针，对吧？但事实上，这个宏里面只有一些奇怪的`{}`和两条语句。事实上，这是一个C语言的GNU扩展被叫做`braced-group within expression`。编译器会对整个语句块进行求值，并用该块中的最后一条语句的值作为返回值。考虑下面的例子，它应该返回`5`。

```C
int x = ({1; 2;}) + 3;
printf("%d\n", x);
```

# `typeof()`

这是一个不标准的GNU的C扩展。这个关键字接受一个参数并返回它的类型。它的完整语意可以在[GCC文档](http://gcc.gnu.org/onlinedocs/gcc/Typeof.html)中找到。

# 对0指针的解引用

对0指针解引用会发生什么呢？是的，这是一个指针的小技巧用以获取成员的类型。它并不会造成程序崩溃，因为这个表达式的本身并不会被求值。所有的编译器都会支持对0的解引用。相同的情形会发生在我们我们要取出某个成员的地址时。编译器并不在意这个值，它仅仅是将成员的偏移地址加到成员所在结构体的地址上而已，在这里结构体的地址是0，然后返回新的地址，即成员在这个结构体内的偏移地址。

```C
struct s {
        char m1;
        char m2;
};
 
/* This will print 1 */
printf("%d\n", &((struct s*)0)->m2);
```

注意，下面的两个定义是相同的：
```C
typeof(((struct s *)0)->m2) c;
char c;
```

# `offsetof(st, m)`

这个宏会返回结构体中某个成员相对于该结构体起始地址的偏移量。它是C标准函数库的一部分（包含在stddef.h中）。在内核中由于没有C标准函数库，但依然包含有这个宏定义。这个宏和我们之前看到的0指针解引用有些许相似，这也避免了使用某些现代编译器的内置函数来实现这个功能。这里是一个杂乱的版本（来自内核./include/linux/stddef.h）：

```C
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```

这个宏会返回在TYPE结构体中叫做MEMBER成员的地址，该结构体存储起始地址为0。

# 将它们结合起来

```C
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:    the pointer to the member.
 * @type:   the type of the container struct this is embedded in.
 * @member: the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({          \
    const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```

如果你认真的看过了本文开始的定义，你会疑惑这个宏定义中的第一句是不是不太对劲。是的，你是正确的。第一行队最终的结果并不会有什么重要作用，它在那儿的原因就是为了类型检查。那么第二行做了什么呢？**它从结构体成员变量的地址中抽取出了该成员在该结构体中的偏移量**用以产生包含它的结构体的地址，就是这样！

在一步一步了解过所有的关键的，具有魔法的操作之后，它原来是如此的简单。

# 参考

- [container_of() explained by Greg KH](http://broken.build/2012/11/10/magical-container_of-macro/)
- [container_of() definition in Linux Kernel](http://lxr.free-electrons.com/source/include/linux/kernel.h#L691)
- [Statements and Declarations in Expressions](https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html#Statement-Exprs)
- [Referring to a Type with typeof](http://gcc.gnu.org/onlinedocs/gcc/Typeof.html)
- [offsetof()](http://en.wikipedia.org/wiki/Offsetof)

---

### ¶ The end
