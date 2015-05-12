---
title: 'Linux内核中do{} while(0)宏定义的意义'
categories:
  - Science & Technology
  - Programming Language
  - C
tags:
  - Linux
  - Linux Kernel
  - C
abbrlink: 75070306
date: 2015-05-12 18:31:00
---

在看Linux源码时，会发现其中的一些宏定义有些冗余，例如：

```C
#define schedstat_set(var, val)        do { var = (val); } while (0)
```

这一句宏定义从直觉上会感觉下面的一句功能相同。

```C
#define schedstat_set(var, val)        var = (val)
```

但事实并非如此。

<!-- more -->

假如我们有如下的宏定义：

```C
#define FOO(x) \
    printf("arg is %s\n", x); \
    do_something_useful(x)
```

然后调用：

```C
if (blah == 2)
    FOO(blah);
```

如果按照宏展开，会发现有一些问题出现了：

```C
if (blah == 2)
    printf("arg is %s\n", blah);
    do_something_useful(blah);
```

但是如果使用如下的宏定义

```C
#define FOO(x) \
    do { \
        printf("arg is %s\n", blah); \
        do_something_useful(blah); \
    } while (0)
```

展开后，就和我们预期的一致了：

```C
if (blah == 2)
    do {
        printf("arg is %s\n", blah);
        do_something_useful(blah);
    } while (0)；
```

> 参考：[http://kernelnewbies.org/FAQ/DoWhile0](http://kernelnewbies.org/FAQ/DoWhile0)

---

### ¶ The end

