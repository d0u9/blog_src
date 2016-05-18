---
title: 'cmake简介(PART III: 变量与宏)'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Cmake
abbrlink: 650586122
date: 2016-05-18 17:06:00
---

在cmake中内置了许多默认的变量，此外还可以根据自己的需要设置变量。cmake还可以根据模板文件自动替换其中的变量，从而实现自动添加版本号等功能。

<!-- more -->

# 变量

- 变量的定义与使用

   在CMakeLists.txt中定义变量使用`set()`命令。

   ```cmake
   set(变量名 值)
   ```

   如果要使用变量，需要用以下方式：

   ```cmake
   ${变量名}
   ```

- 内置变量

   cmake内置了很多变量，这些变量可以直接拿来使用，具体的可以使用**man**命令来查看。

   ```cmake
   man 7 cmake-variables
   ```

# 模板替换

在实际的项目中，我们一般需要定义一个宏来说明当前的版本号。

在camke中，我们可以通过定义模板，在替换其中的指定部分来针对不同的编译参数使用不同的变量：

首先我们创建模板，`cmake/config.h.in`：

```C
#define PROJECT_VERSION_MAJOR @VERSION_MAJOR@
#define PROJECT_VERSION_MINOR @VERSION_MINOR@
```

其中的`@...@`部分会被替换。

之后我们在**CMakeLists.txt**中添加如下内容：

```cmake
# 设置变量名
set (VERSION_MAJOR 2)
set (VERSION_MINOR 0)

# 复制config.h.in到config.h，并替换其中@中间的变量
configure_file (
	"${PROJECT_SOURCE_DIR}/cmake/config.h.in"
	"${PROJECT_SOURCE_DIR}/include/config.h"
	)
```

`configure_file()`会将**config.h.in**中的内容用之前设定的变量所替换，并将替换后的内容保存在**include/config.h**中。

# 宏变量

很多情况下，我们需要根据条件来编译不同的模块。这里假设我们需要根据实际情况来选择是否要输出`Hello World!`。

我们可以设置一个叫做`DUMP_HELLO`的宏变量，来控制是否需要打印。

修改**main.c**如下：

```C
#include <stdio.h>
#include "include/config.h"
#ifdef DUMP_HELLO
#include "include/echo.h"
#endif
#include "include/operations.h"

int main(int argc, char *argv[]) {
        printf("The current of hello is %d.%d\n",
               PROJECT_VERSION_MAJOR, PROJECT_VERSION_MINOR);

#ifdef DUMP_HELLO
        echo();
#endif

        int a = 1, b = 2;
        int c = add(a, b);
        printf("c = %d\n", c);

        return 0;
}
```

我们可以通过修改模板**config.h.in**来控制是否定义该宏，在**config.h.in**中添加如下内容：

```C
#cmakedefine DUMP_HELLO
```

并在**CMakeLists.txt**中添加：

```cmake
option (DUMP_HELLO
	"print Hello World when execute" OFF)
```

注意这里我们设置其为`OFF`也就是不打印`Hello World！`。

既然不打印了，那也就不需要在编译**echo.c**这个文件了，我们可以在**lib/CMakeLists.txt**中修改如下：

```cmake
if (DUMP_HELLO)
	add_library(echo echo.c)
endif (DUMP_HELLO)
```

这样就可以了。

---

### ¶ The end

