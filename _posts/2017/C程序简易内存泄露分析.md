---
title: 'C程序简易内存泄露分析'
categories:
  - Science & Technology
  - Programming Language
  - C
tags:
  - C
abbrlink: 851584278
date: 2017-03-03 10:35:00
---

最近的项目中遭遇了内存泄露。起初尝试用人脑阅读并分析整个内存分配/回收的代码逻辑，但以失败告终。最后不得不使用脚本对代码进行了分析，找出了泄漏点。

<!-- more -->

# 操作过程

1. 找出所有的内存分配和释放的位置：

   在源文件中，使用**cscope**工具或**grep**找出所有的内存分配和释放位置。

2. 编写内存分配/释放封装宏。

   这个宏里使用了GNU GCC的扩展宏特性，测试后发现clang编译器对该特性也支持。
   但没有在Vistual Studio环境下测试，是否支持不详。

   ```C
   #include <string.h>
   #include <stdlib.h>
   #include <stdio.h>

   #define __stringify_1(x...)     #x
   #define __stringify(x...)       __stringify_1(x)


   #define calloc(m,n)     ({                                          \
                               char *p = (char*)calloc(m,n);           \
                               fprintf(stderr,                         \
                               __FILE__ ":" __stringify(__LINE__)      \
                               " alloc: %p\n", p);                     \
                               (void*)p;                               \
                           })

   #define free(p)         ({                                          \
                               fprintf(stderr,                         \
                               __FILE__ ":" __stringify(__LINE__)      \
                               " free: %p\n", p);                      \
                           })
   int main(void)
   {
       void *z = calloc(1, 16);
       free(z);
   }
   ```

3. 运行代码，并将标准错误重定向至某输出文件。

   ```
   test.c:25 alloc: 0x244f010
   test.c:27 free: 0x244f010
   ```

4. 对输出进行分析。

# 分析样例

下图是有内存发生泄露时，分析输出的结果：

![](http://oyui6c341.bkt.clouddn.com/images/2017/C程序简易内存泄露分析/01.jpg)

从图中可以看出，`src/socket.c`文件的53行分配了12922次，但并没有与之对应释放。 通过分析排查，修该了该处问题后，重新运行上述分析过程：

![](http://oyui6c341.bkt.clouddn.com/images/2017/C程序简易内存泄露分析/02.jpg)

从上图可以看出，该处的泄露已经不存在。

---

### ¶ The end


