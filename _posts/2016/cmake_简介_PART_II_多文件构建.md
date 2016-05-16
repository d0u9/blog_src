---
title: 'cmake简介(PART II: 多文件构建)'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Cmake
abbrlink: 841494507
date: 2016-05-16 19:13:00
---

在实际的项目中，往往会有许多不同的子目录，用来保存项目不同模块的源码。使用cmake可以方便的管理这些分布在不同文件夹中的源码。

本文会使用cmake对多个不同文件夹下的c文件进行编译，最后再链接成二进制执行文件。

<!-- more -->

# 项目文件结构

这次用到的文件目录结构如下：

```bash
tree -L 2
```

```
.
├── build
├── CMakeLists.txt
├── include
│   ├── echo.h
│   └── operations.h
├── lib
│   ├── CMakeLists.txt
│   ├── echo.c
│   └── operations.c
└── main.c
```

# 目录说明

在该项目中，`main()`函数保存在**main.c**文件中。**main.c**文件通过`#include`来引用**include/operations.h**头文件和**include/echo.h**。

在**include/operations.h**头文件中，声明了一个`int add(int, int)`函数，该函数打印一行字符并返回两个整数的和。`int add(int, int)`的具体实现在**lib/operations.c**中。

在**include/echo.h**头文件中，声明了一个`void echo(void)`函数，该函数用来代替之前`main`函数中打印**Hello World**的功能，具体的定义保存在**lib/echo.c**中。

在cmake中，每一个包含有源文件的子目录都可以被当做一个子项目，因此每一个子目录也需要包含**CMakeLists.txt**文件用来说明当前子目录的情况。

子目录中的**CMakeLists.txt**会共享其父目录中的变量和设置。

# 新加入的文件

- **include/echo.h**

   ```C
   #ifndef ECHO_H__
   #define ECHO_H__

   void echo(void);

   #endif
   ```

- **include/operations.h**

   ```C
   #ifndef OPERATIONS_H__
   #define OPERATIONS_H__

   int add(int, int);

   #endif
   ```

- **lib/echo.c**

   ```C
   #include <stdio.h>
   #include "../include/echo.h"

   void echo(void) {
           printf("Hello World!\n");
   }
   ```

- **lib/operations.c**

   ```C
   #include <stdio.h>
   #include "../include/operations.h"

   int add(int a, int b) {
           printf("Add opertion!\n");
           return a + b;
   }

   int add(int, int);
   ```


# CMakeLists.txt文件

- **lib**中的**CMakeLists.txt**

   **lib**下的**CMakeLists.txt**内容很简单，仅包含下面一行：

   ```cmake
   # 增加一个叫做**lib**的库目标（library target），并且这个库由**operations.c**和**echo.c**两个源文件生成。
   add_library (lib operations.c echo.c)
   ```

- 修改项目根目录下的**CMakeLists.txt**。

   在根目录下的**CMakeLists.txt**中需要说明哪些目录是包含头文件的，哪些目录包含源文件。

   添加如下内容：

   ```cmake
   # 用来构建该项目所用cmake的最低版本。
   cmake_minimum_required (VERSION 2.6)

   # 项目名
   project (Hello_World)

   # 编译器用来寻找include文件的目录
   include_directories ("${PROJECT_SOURCE_DIR}/include")

   # 添加包含源文件的子目录
   add_subdirectory ("${PROJECT_SOURCE_DIR}/lib")

   # 使用main.c来生成hello。
   add_executable (hello main.c)

   # 链接target所需要的库
   target_link_libraries (hello lib)
   ```

# 构建项目

构建方法可以前一样：

```bash
cd build
cmake ..
make
./hello
```

输出如下:

```
Hello World!
Add opertion!
c = 3
```

# 分开编译不同的源文件

之前使用`add_library (lib operations.c echo.c)`是将operations.c和echo.c一起编译成了**liblib.a**，然后在链接到最后的**hello**二进制文件中。

但是有时候，需要将不同的源文件编译成不同的库文件然后按需链接。这时需要修改**lib/CMakeLists.txt**为如下内容：

```cmake
add_library(echo echo.c)
add_library(operations operations.c)
```

并且修改顶层**CMakeLists.txt**中的`target_link_libraries()`为如下内容：

```cmake
target_link_libraries (hello echo operations)
```

从而告诉cmake`hello`需要`echo`和`operations`两个库。

之后删除**build**文件夹内的内容，重新构建就好了。

---

### ¶ The end
