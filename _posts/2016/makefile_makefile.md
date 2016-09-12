---
title: 'Makefile? Makefile!'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Make
abbrlink: 412858525
date: 2016-09-12 13:49:00
---

Makefile的写法前前后后也看了很多遍，虽然大概知道怎么回事，但每次写稍微复杂一点（多文件编译，将中间文件编译到build目录中防止文件污染）的Makefile还是会有些不知所措。今天稍微总结下，方便以后来找记忆。

<!-- more -->

# 示例代码

示例代码异常简单：

```C
// main.c
#include "foo.h"

int main(void)
{
        int a = 100;
        int b = foo(a);
        return b;
}

// foo.h
#ifndef __FOO_H
#define __FOO_H

extern int foo(int k);

#endif

// foo.c
#include <stdio.h>
#include "foo.h"

int foo(int k)
{
        printf("K = %d\n", k);
        return (k + 1);
}
```

这段代码的功能一目了然，这里就不多做解释了。

# Step1: 基本功能实现版

```make
CC = gcc
CFLAGS = -Wall -g -O1 -I.

BIN = sample

BUILD_DIR = ./build

SRCS = $(wildcard *.c)
OBJS = $(SRCS:%.c=$(BUILD_DIR)/%.o)

all:
	mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) $(SRCS) -o $(BIN)

.PHONY: clean
clean:
	rm -fr build *.o *.d
```

这个版本会将生成的中间文件（这里主要是.o文件）保存在**build**文件夹下，从而避免对源码树的污染。

`OBJS = $(SRCS:%.c=$(BUILD_DIR)/%.o)`表示，将`SRCS`变量中的`.c`替换为`$(BUILD_DIR)`。

`SRCS = $(wildcard *.c)`表示使用通配符匹配所有的`.c`文件。

# Step2: 具有自动识别文件改动功能的版本

```make
CC = gcc
CFLAGS = -Wall -g -O1 -I.

BIN = sample

BUILD_DIR = ./build

SRCS = $(wildcard *.c)
OBJS = $(SRCS:%.c=$(BUILD_DIR)/%.o)

all: $(OBJS)
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) $^ -o $(BIN)

$(BUILD_DIR)/%.o: %.c
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) -c $< -o $@

.PHONY: clean
clean:
	rm -fr build *.o *.d
```

这里有几点要解释：

- `@mkdir -p $(BUILD_DIR)`, 这里的`@`表示，不打印当前这条命令。默认情况下，**make**执行命时，会自动输出当前执行的指令。
- `$^`变量，这个变量表示当前生成目标下（例如all这个目标），所有的依赖名。
- `$<`变量，这个变量表示当前生成目标下（例如all这个目标），第一个依赖的名字。
- `$@`变量，这个变量表示当前生成目标的名字。


# Step3: 具有识别头文件改动功能的版本

```make
CC = gcc
CFLAGS = -Wall -g -O1 -I.

BIN = sample

BUILD_DIR = ./build

SRCS = $(wildcard *.c)
OBJS = $(SRCS:%.c=$(BUILD_DIR)/%.o)
DEPS = $(OBJS:%.o=%.d)

all: $(OBJS)
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) $^ -o $(BIN)

-include $(DEPS)

$(BUILD_DIR)/%.o: %.c
	@mkdir -p $(BUILD_DIR)
	$(CC) $(CFLAGS) -MMD -c $< -o $@

.PHONY: clean
clean:
	rm -fr build *.o *.d
```

这一步我们加入了`-MMD`编译选项，变量`DEPS = $(OBJS:%.o=%.d)`和`-include $(DEPS)`。

对于`-MMD`选项可以自行查询**gcc**的man手册。简单的说就是gcc会根据预处理的结果生成`.d`文件，`.d`文件是**make**兼容的依赖依赖树。通过**make**的`-include`可以引入该依赖，从而避免的对头文件的处理。

这里的参考来源：[http://stackoverflow.com/a/30142139/3824053](http://stackoverflow.com/a/30142139/3824053)


## Step4: 小优化

```make
CC = gcc
CFLAGS = -Wall -g -O1 -I.

BIN = sample

BUILD_DIR = ./build

SRCS = $(wildcard *.c)
OBJS = $(SRCS:%.c=$(BUILD_DIR)/%.o)
DEPS = $(OBJS:%.o=%.d)

all: $(BUILD_DIR)/$(BIN)

$(BUILD_DIR)/$(BIN): $(OBJS)
	@mkdir -p $(@D)
	$(CC) $(CFLAGS) $^ -o $@

-include $(DEPS)

$(BUILD_DIR)/%.o: %.c
	@mkdir -p $(@D)
	$(CC) $(CFLAGS) -MMD -c $< -o $@

.PHONY: clean
clean:
	rm -fr build *.o *.d
```

修改`@mkdir -p $(BUILD_DIR)`到`@mkdir -p $(@D)`。`$(@D)`是**make**的内置变量，从[手册中](http://www.chemie.fu-berlin.de/chemnet/use/info/make/make_15.html)可以找到其描述为*The directory part and the file-within-directory part of $@*.

修改了`all`目标，这样可以避免不必要的链接操作。


# 为不同的构建目标使用不同的编译参数

有时候我们为了测试需要，需要在编译时添加某些参数，例如`-g`参数。这时我们希望能用不同的目标来生成不同的结果。那么可以这么做（这个例子和上面的例子没有关系，只是为了表明如何使用与目标绑定的变量）


```make
release: DEBUG_FLAGS = -O2
release: build

build:
	gcc $(DEBUG_FLAGS) *.c -o out

debug: DEBUG_FLAGS = -g -O1 -DDEBUG
debug: build
```

这时当我们执行`make`和`make debug`时就可以分别以不同的参数来编译了。


# 添加新的文件扩展后缀

有时候会用到某些扩展名不常见的文件类型，例如使用**m4**进行替换的模板文件`*.in`。这样的文件不能被**make**直接的识别，需要我们手动的添加这些扩展名。

```make
.SUFFIXES: .h.in .h

%.h: %.h.in
        $(M4) $(M4_SCRIPT) $^ > $@
```

---

### ¶ The end
