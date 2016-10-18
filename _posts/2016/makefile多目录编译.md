---
title: 'Makefile多目录编译'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Make
abbrlink: 884470221
date: 2016-10-18 15:38:00
---

在真实的项目中，不同的源文件往往按照其所属模块，文件类型等分别放在不同的子目录中。一个典型的例子是会将所有的`.h`文件放在**headers**文件夹内，而所有的`.c`文件则被放置于**src**文件夹内。此外，一个项目可能会调用某些子库，这些库往往会被编译成动态或者静态的链接文件，在最后的链接阶段被链接进二进制文件中。在本文中，我们将用两个例子对这两种情况（直接编译和分别编译成库最后链接）进行说明。

<!-- more -->

# 直接对不同目录内的文件进行编译

测试用目录结构如下：

```
.
├── headers
│   ├── src1.h
│   └── src2.h
├── Makefile
└── srcs
    ├── main.c
    ├── src1.c
    └── src2.c
```

我们将所有的`.c`文件都放置在了**srcs**目录中，而头文件都放置在了**headers**目录中。其中，在**src1.c**和**src2.c**中分别定义了一个函数，通过头文件导出后，供**main.c**使用。

Makefile内容如下：

```make
CC = gcc
BIN = demo.out

CFLAGS += -std=c99 -Wall -g -fPIC -Iheaders
BUILD_DIR = build

SRCS = $(wildcard srcs/*.c)
HEDS = $(wildcard headers/*.h)
OBJS = $(patsubst %.c, $(BUILD_DIR)/%.o, $(notdir $(SRCS)))
DEPS = $(patsubst %.o, %.d, $(OBJS))

VPATH = srcs:headers

.PHONY: all
all: _PRE $(BIN)
	@echo "Start compiling"

-include $(DEPS)

$(BIN): $(OBJS)
	$(CC) $(CFLAGS) -o $(BIN) $^

$(BUILD_DIR)/%.o: %.c
	$(CC) $(CFLAGS) -MMD -c -o $@ $<

.PHONY: _PRE
_PRE:
	mkdir -p $(BUILD_DIR)

.PHONY: clean
clean:
	rm -fr build
```

其中关键处为`VPATH = srcs:headers`。VPATH是一个用冒号分割的目录列表，列表内的目录均为编译时进行源文件搜索的目录。举例来说，我们在项目根目录下运行`make`命令后，make会在当前项目根目录下寻找**main.c**，**src1.c**，**src2.c**以及对应的头文件，当无法找到时，会继续搜索**srcs**和**headers**两个目录。这也意味着，这些目录中的文件**不能重名**。

如果想要更加细致的控制包含不同文件的目录，例如仅在**headers**文件夹中搜索`.h`文件，而`.c`文件则在**srcs**中进行搜索，则可以使用`vpath`命令（注意是小写）。

如此，可将上面例子中的

```make
VPATH = srcs:headers
```

替换为

```make
vpath %.h headers
vpath %.c srcs
```

此外还需要注意的是，文件的搜索是有先后顺序的。具体的顺序可以看[GNU make 手册](https://www.gnu.org/software/make/manual/make.html)。

# 将子目录中的文件编译成库，之后进行链接

项目目录如下：

```
.
|-- lib
|   |-- lib.c
|   |-- lib.h
|   `-- Makefile
|-- main.c
`-- Makefile
```

其中，**lib**文件夹下为单独的库项目，它包括一个**Makefile**文件。在**lib**文件夹中执行`make`，会生成**.so**和**.a**两个库文件。

**main.c**会用到**lib**文件夹中的库文件，生成最终的二进制文件。

## lib库文件的Makefile

Makefile内容如下：

```make
CC = gcc
AR = ar
LIB_A = liblib.a
LIB_SO = liblib.so

BUILD_DIR = build

SRCS = $(wildcard *.c)
OBJS = $(patsubst %.c, $(BUILD_DIR)/%.o, $(SRCS))
DEPS = $(patsubst %.o, %.d, $(OBJS))

.PHONY: all
all: $(OBJS)
	$(CC) -fPIC -shared -o $(LIB_SO) $^
	$(AR) -cvq $(LIB_A) $^

-include $(DEPS)

$(BUILD_DIR)/%.o: %.c
	$(CC) -fPIC -MMD -c -o $@ $<

$(OBJS): | $(BUILD_DIR)

.PHONY: $(BUILD_DIR)
$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

.PHONY: clean
clean:
	rm -fr $(BUILD_DIR)
	rm -fr *.a
	rm -fr *.so
```

这个**Makefile**比较简单，主要生成**liblib.a**和**liblib.so**两个库文件。

## 顶层目录的Makefile

```make
CC = gcc
AR = ar
BIN = test.out

BUILD_DIR = build
SUB_LIB = lib

SRCS = $(wildcard *.c)
OBJS = $(patsubst %.c, $(BUILD_DIR)/%.o, $(SRCS))
DEPS = $(patsubst %.o, %.d, $(OBJS))

.PHONY: all
all: $(SUB_LIB) $(OBJS)
	$(CC) -Ilib -o $(BIN) $(OBJS) $(SUB_LIB)/*.a

-include $(DEPS)

$(BUILD_DIR)/%.o: %.c
	$(CC) -Ilib -fPIC -MMD -c -o $@ $<

$(OBJS): | $(BUILD_DIR)

.PHONY: $(BUILD_DIR)
$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)
	
.PHONY: $(SUB_LIB)
$(SUB_LIB):
	make CC=$(CC) AR=$(AR) -C $(SUB_LIB)

.PHONY: clean
clean:
	rm -fr *.out
	rm -fr build
	make -C $(SUB_LIB) clean
```

在这个Makefile文件中，需要注意的地方有在于

```make
.PHONY: $(SUB_LIB)
$(SUB_LIB):
	make CC=$(CC) AR=$(AR) -C $(SUB_LIB)
```

这个target会执行`make -C`，其中`-C`的含义如下：

> -C dir, --directory=dir
> Change  to  directory dir before reading the makefiles or doing anything else.
> If multiple -C options are specified, each is interpreted relative to the previous one: -C / -C etc is equivalent to -C /etc. This is typically used with recursive invocations of make.

简单的说，就是递归的执行make -- 进入子文件夹**lib**中，并且执行`make`。

我们可以向递归的`make`命令传送参数，例子中我们将`$(CC)`也就是`gcc`赋值给了子目录中的`CC`变量，这样当我们想要使用`clang`而非`gcc`的时候只需要在顶层的`Makefile`中修改`CC`变量即可。

此外，在进行clean的时候，我们也可以递归的进行：

```make
.PHONY: clean
clean:
	rm -fr *.out
	rm -fr build
	make -C $(SUB_LIB) clean
```

# 关于创建build文件夹

## 方法1

一般我们希望将生成的所有中间文件（.o，.d等）放在一个特定的文件夹中（一般是build），这样可以方便我们管理，避免对源文件目录不必要的污染。

在之前，我在写Makefile时，如果要创建**build**文件夹，一般的做法是单独写一个target，将其放在所有所有依赖的最前面。

但后来在stackoverflow上看到有人讲这样是不正确的，因为如果在编译时使用`-j`指定了多线程编译，那么这个顺序就不能确定了。

为了保证**build**文件夹在所有的动作之前创建，我们可以使用_order-only-prerequisites_（见GCC官方文档Types of Prerequisites节）。

## 方法2

使用`$(shell )`函数可以方便的执行shell命令，我们可以在Makefile刚开始执行时，使用该函数创建`build`文件夹。

```make
build_abs_path := $(shell mkdir -p build && cd build && pwd)
```

---

### ¶ The end
