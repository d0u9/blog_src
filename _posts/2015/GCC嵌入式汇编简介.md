---
title: GCC嵌入式汇编简介
categories:
  - Science & Technology
  - Programming Language
  - Assembly Language
tags:
  - GCC
  - Assembly Language
abbrlink: 3564505605
date: 2015-12-22 17:43:00
---

虽然对汇编有些许了解，也能半知半解的看懂C潜入的汇编代码，但是每次都有一种隔靴搔痒的感觉。遂总结之。

<!-- more -->

# 基本内联汇编

最基本的内联汇编格式有些像一个函数的调用：

```Assembly
asm( asm-code-strings );
```

需要注意的是，`asm-code-string`是一个字符串，在这个字符串中的代码会被GCC直接插入到中间生成汇编代码中，并被链接到最后的可执行文件中。所以，你需要注意报错的是不是汇编器。

举个例子：

```C
// assembly_1.c
int foo(void)
{
	asm( "movl $0, %eax	\n\t"
	     "addl $1, %eax"
	   );
}
```

在Mac OSX 10.11.2 下使用`gcc assembly_1.c -S -fno-asynchronous-unwind-tables`命令进行编译后可得到如下的汇编文件：

```Assembly
## assembly_1.c
	.section	__TEXT,__text,regular,pure_instructions
	.macosx_version_min 10, 11
	.globl	_foo
	.align	4, 0x90
_foo:                                   ## @foo
## BB#0:
	pushq	%rbp
	movq	%rsp, %rbp
	## InlineAsm Start
	movl	$0, %eax
	addl	$1, %eax
	## InlineAsm End
	xorl	%eax, %eax
	popq	%rbp
	retq


.subsections_via_symbols
```
在其中能看到，我们插入的内联汇编被标记了出来。

# `asm` 关键字的扩展写法

GCC提供了一种扩展写法，能够让程序员将C的变量和汇编中的操作数对应起来，哪些寄存器会被嵌入的汇编代码所改写。在这种情况下，GCC能够更好保护那些会被改写的寄存器，将汇编代码和C代码更好的融合。

通用的嵌入式汇编写法格式如下：

```C
asm [volatile] (
	code-string
	: output-list		/* 可选的 */
	: input-list		/* 可选的 */
	: overwrite-list	/* 可选的 */
);
```

volatile的作用是防止GCC编译器自己做主将汇编代码优化掉，如果在asm语句之后的C代码没有使用任何output-list中的输出，那么需要用volatile来防止编译器对该asm代码段的优化。

`code-string` 汇编代码。需要注意的是，如果需要嵌入多个汇编指令，需要使用换行符进行分割，约定俗称的分隔符是`\n\t`。
`output-list` 汇编代码的输出映射。
`intput-list` 汇编代码的输入映射。
`overwrite-list` 会被嵌入的汇编代码改写的寄存器列表。

## output-list

output-list由逗号分割不同的段，每个段的格式为`[name] constraint (expr)`。

- name仅仅是一个符号，用来表示汇编代码中的某个操作数。无论是在string-list中还是output-list中，它必须被方括号包裹。命令方式和C语言一致，任何在C中合法的变量名都可以被使用，即使是在函数中定义过的变量名也可以使用，该操作数的作用域仅仅是声明该操作数的asm代码块。name可以被省略，当它被省略的时候，会按照出现顺序进行编号。

- constraint的作用主要是用来说明输出的位置是内存还是寄存器还是什么位置，具体的constraint见[constraints](#constraints)。

- expr是一个C表达式表明指令结果的存储位置，该表达式必须为左值表达式。需要注意的是，圆括号是必须的。

## input-list

input-list与output-list类似，也是由逗号分隔的段，每个段表示一个操作数的映射,格式为`[name] Constraint (expr)`。

- name与output-list类似，唯一的不同在于，如果name省略的话，编号并不会从0开始，而是从output-list中操作数的个数开始。例如，如果在output-list中有3个操作数，不管他们有没有name，都会从3开始。

- constraint的作用见[constraint](constraint)。

- expr是一个c表达式，它的值作为汇编代码对应操作数的值。

## overwrite-list

用逗号分隔的，会被嵌入的汇编代码改写的寄存器，寄存器名字需要放在双引号中。比较常用的有一个`memory`参数，它告诉编译器这段汇编代码除了input-list和output-list中指明的操作数之外，还对内存进行的读或者写。这样GCC执行这段汇编代码之前会将内存刷新，同时在汇编执行外之后也会刷新对应的寄存器，确保数据的一致。

## constraints <a id="constraints"></a>

constraints表明了操作数的存储位置和存储方式，比较常用的见下表：

| Simple Constraints   |                                            |
| -------------------- | ------------------------------------------ |
| m                    | 内存位置                                   |
| r                    | 通用寄存器                                 |
| i                    | 立即数，0..0xffffffff                      |
| g                    | 通用寄存器，立即数或内存，编译器自己决定   |
| a,b,c,d,S,D          | 分别表示eax,ebx,ecx,edx,esi,edi 寄存器     |


| Constraint Modifier Characters   |                                                                        |
| -------------------------------- | ---------------------------------------------------------------------- |
| =                                | 表明操作数被新值写入                                                   |
| +                                | 操作数会被读取并写入                                                   |
| &                                | 该寄存器仅被用于输出，主要防止某些寄存器在输入后被重用，导致数据错误   |
| %                                | 该操作数和下一个操作数允许交换，用以提升性能                           |

## 例子

```C
#include <stdio.h>

int get_year(int a, int *b, char * c)
{
        long t2;
        int ret;

        asm (
                "addl %[b], %0                                  \n\t"
                "xorq %%rdx, %%rdx                              \n\t"
                "movl $58, %%edx                                \n\t"
                "movb %%dl, 0x04(%1)                            \n\t"
                : "=r" (ret), "=&D" (t2)
                : "0" (a), [b] "erm" (*b), "1" (c)
                : "edx", "memory"
        );

        return ret;
}

int main(void)
{
        int a = 15, ret;
        int *out = &a;
        char string[] = "Year  ";
        ret = get_year(2000, out, string);
        printf("%s%d\n", string, ret);
        return 0;
}
```

这只是一个例子，所以里面有很多可以被优化的地方。

这段代码的作用是将两个数2000和15相加，之后为字符串"Year  "添加冒号。

2000和15两个数分别用传值和地址传递两种方式实现。

需要注意的是第13行的`"=&D"`，这里使用&来表示保存t2的寄存器不要被重用。由于编译器会存在某些优化机制，导致为t2保留的寄存器在返回之前暂时被用作其他的目的，从而造成某些错误。

这里t2只是一个占位符，最后会被编译器优化掉。这种写法来自于Linux[内核的memcpy函数](https://github.com/torvalds/linux/blob/master/arch/x86/boot/compressed/string.c)

# 关于&的一点说明

使用`&`表示该输出操作数不能和输入操作数重叠。如果不指定`&`，GCC编译器可能会在输出该操作数指定的寄存器之前，将该寄存器用以其他的目的，造成该操作数被覆盖。

在[这篇文章中](http://www.nongnu.org/avr-libc/user-manual/inline_asm.html)作者给出了一个不错的例子：

```C
asm volatile("in %0,%1" "\n\t"
    "out %1, %2" "\n\t"
    : "=&r" (input)
    : "I" (_SFR_IO_ADDR(port)), "r" (output)
);
```

在上面的这段代码中，input会被用来存放从IO输入的数据。由于汇编代码中，IO的输入在输出之前，那么在第一条指令执行完后，`%0`保存了我们从IO接受的数据，如果不加`&`的话，那么在吓一条指令中，编译器可能会继续使用`%0`所指定的寄存器用来保存输出数据，因为编译器认为上一条指令执行完后，`%0`就可以释放了。

`&`加入之后，可以保证`%0`所用的寄存器不回再被再次使用。

# 参考

1. [GCC manual - How to Use Inline Assembly Language in C Code](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html#Using-Assembly-Language-with-C)

2. [GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)

3. [Stackoverflow - llvm reports: unsupported inline asm: input with type 'void *' matching output with type 'int'](http://stackoverflow.com/questions/34446928/llvm-reports-unsupported-inline-asm-input-with-type-void-matching-output-w)

4. [Inline Assembler Cookbook](http://www.nongnu.org/avr-libc/user-manual/inline_asm.html)

---

### ¶ The end
