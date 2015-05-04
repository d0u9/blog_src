---
title: '64bit cpuid汇编中为何需要sub $8, %rsp'
categories:
  - Science & Technology
  - Programming Language
  - Assembly Language
tags:
  - Assembly Language
  - 64bit
abbrlink: 3018250366
date: 2015-05-04 11:49:00
---

想要将《Professional Assembly Language》中87页使用printf来打印Vendor ID信息代码改写成64位的。但由于64位函数调用约定与32位函数调用约定不同，遇到了一些问题。

在32bit程序中，函数的调用传参是靠栈完成的，通过`push`与`pop`的配合，可以将caller函数的参数传递给callee。但是在64bit的程序中，由于寄存器的个数增加。传参方式也有所改变。

<!-- more -->

# 64bit函数调用的传参

在64bit程序中，函数的参数传递主要依靠寄存器完成。这主要是因为支持64位的处理器相对仅支持32位的处理器多了若干个寄存器。这就使得大多数情况下的函数调用可以不经过栈。

这一点在CSAPP中被提到。下面引自CSAPP-316页-procedures：

> By doubling the register set, programs need not be so dependent on the stack for storing and retrieving procedure information.

同时，CSAPP也说明了，哪些寄存器用来保存第几个参数。具体见下表:


| size   | 1      | 2      | 3      | 4      | 5      | 6      |
| :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| 64     | %rdi   | %rsi   | %rdx   | %rcx   | %r8    | %r9    |
| 32     | %edi   | %esi   | %edx   | %ecx   | %r8d   | %r9d   |
| 16     | %di    | %si    | %dx    | %cx    | %r8w   | %r9w   |
| 8      | %dil   | %sil   | %dl    | %cl    | %r8b   | %r9b   |

也就是说，取出一个函数的参数，判断是多少bit的长度，之后保存在对应的寄存器中就可以了。

# 使用printf打印处理器的Vendor ID信息。

代码如下：

```Assembly
＃as cpuid2_64.s -o cpuid2_64.o
＃ld -macosx_version_min 10.9 -lSystem cpuid2_64.o
.data
output:
    .asciz "The processor Vendor ID is '%s'\n"
.static_data
    .lcomm buffer, 12          ＃分配静态存储
.text
.globl  _main
_main:
    movq    $0, %rax
    cpuid
    movl    $buffer, %edi
    movl    %ebx, (%edi)
    movl    %edx, 4(%edi)
    movl    %ecx, 8(%edi)
    subq    $8, %rsp
    lea output(%rip), %rdi                    #传入第一个参数
    lea buffer(%rip), %rsi                    #传入第二个参数
    callq   _printf
    movq    $0, %rdi                          ＃退出程序
    movq    $0x2000001, %rax
    syscall
```

对于参数的传递，很好理解，但是我自己编写的第一版程序没有第22行的`subq $8, %rsp`。
如果不加入这一行会报段错误，程序不能正常执行。但是为何要加这一行一直不明白，经过各种百度，谷歌依然没有找到结果。最后仔细的看了CSAPP的第319页，发现了如下的一段：

> Whenever one function (the caller) calls another (the callee), the return address gets pushed onto the stack. By convention, we consider this part of the caller’s stack frame, in that it encodes part of the caller’s state.

至此，终于能够理解为何要`subq $8, %rsp`了，原来这8个字节是用来在函数调用的时候保存返回地址用的。

---

### ¶ The end

