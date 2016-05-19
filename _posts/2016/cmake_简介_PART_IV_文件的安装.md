---
title: 'cmake简介(PART IV: 文件的安装)'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Cmake
abbrlink: 1034170720
date: 2016-05-19 16:17:00
---

项目构建好之后，需要进行安装，使用cmake可以很方便的将不同类型的文件安装到不同的文件夹下。在本文中，我们要将生成的二进制文件安装到目标目录下的**bin**中，**.h**文件安装到**include**文件夹下，而生成的两个**.a**文件安装到**lib**中。

<!-- more -->

# 项目文件结构

这次的项目文件结构和之前的相同：

```
.
├── build
├── cmake
│   └── config.h.in
├── CMakeLists.txt
├── include
│   ├── config.h
│   ├── echo.h
│   └── operations.h
├── lib
│   ├── CMakeLists.txt
│   ├── echo.c
│   └── operations.c
└── main.c
```

# `CMAKE_INSTALL_PREFIX`变量

`CMAKE_INSTALL_PREFIX`变量是cmake的一个内置变量，它的作用和使用传统`./configure`安装时的`--prefix=<path>`作用是相同的，即指定文件的安装位置，在默认的情况下，该变量的位置是`/usr/local/`。

如果在安装时，要制定到其他位置，需要使用

```bash
cmake -DCMAKE_INSTALL_PREFIX:PATH=<path>
```

# 操作过程

- 二进制文件和头文件的安装

   我们最终生成的二进制文件名为**hello**，我们期望将它安装在**bin**目录中，那么需要在项目根目录下的**CMakeList.txt**中添加如下一行：

   ```cmake
   install (TARGETS hello DESTINATION bin)
   ```

   其中`TARGETS`为固定，`TARGETS`后面跟多个文件，这些文件将被安装到`DESTINATION`指定的文件夹中，也就是**bin**文件夹。这里的`DESTINATION`也是固定用法。

   安装头文件与此类似：

   ```cmake
   install (FILES "${PROJECT_SOURCE_DIR}/include/config.h" DESTINATION include)
   ```

   不过这里使用`FILES`而不是<del>`TARGETS`</del>。这里我们将项目目录下`include/config.h`文件安装到**include**文件夹中。

- 库文件的安装

   由于我们的库文件在**lib**文件夹下，因此对于库文件的安装可以把`install()`命令写在**lib**文件夹下的**CMakeList.txt**中。

   在**lib/CMakeLists.txt**修改为如下内容：

   ```cmake
   if (DUMP_HELLO)
       add_library(echo echo.c)
       install (TARGETS echo DESTINATION lib)
   endif (DUMP_HELLO)

   add_library(operations operations.c)

   install (TARGETS operations DESTINATION lib)
   ```

# 重新编译安装

假如我们要安装到**/tmp/hello**，那么执行：

```bash
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/tmp/hello ..
make
make install
```

使用`tree`命令可以看到：

```
/tmp/hello
├── bin
│   └── hello
├── include
│   └── config.h
└── lib
    ├── libecho.a
    └── liboperations.a
```

---

### ¶ The end

