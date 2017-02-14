---
title: 'Examples for building VS project via cmake'
categories:
  - Science & Technology
  - Tools
tags:
  - Tools
  - Cmake
  - Visual Studio
abbrlink: 1370660941
date: 2017-02-15 03:47:00
---

I do dislike Windows OS, however, sometimes using Windows OS is inevitable. As a command line fans, rather than drag and click mouse on the windows, I prefer to use keyboard mostly, even when using Visual Studio IDE.

<!-- more -->

# Cmake Tool

As dictated on cmake's homepage:

> Cmake is an __open-source, cross-platform__ family of tools designed to build, test and package software. Cmake is used to control the software compilation process using simple platform and compiler independent configuration files, and generate native makefiles and workspaces that can be used in the compiler environment of your choice.

Actually, Cmake is a powerful tool which permits its users to create platform independent projects. More importantly, Cmake can be configured via GUI and text file that gives users a great convenience.

# Goal

In this example, we will create a simple Visual Studio solution which contains a dynamic linked library (dll) project and an test project. The test project will invoke the functions which are provided by dll project to test its correctness.

The full solution can be found on my Github page:

[https://github.com/d0u9/examples/tree/master/Windows/cmake/create_dll_and_its_test_project](https://github.com/d0u9/examples/tree/master/Windows/cmake/create_dll_and_its_test_project)

# Details

## The solution

The solution targets at providing a dll library, named **my_lib**, which can be linked in by the third part software. Also, for testing purpose, this solution includes a testing project, named **test_project**, to test the dll's functionalities and correctness.

In this example, the dll library exports two functions, the first adds up two integers and the second just prints out a line of "Hello World". The test project invokes these two exported functions and tests their correctness.

## The main CMakeLists.txt

```cmake
cmake_minimum_required (VERSION 3.6)

project (create_dll_and_its_test_project)

set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

include_directories ("${PROJECT_SOURCE_DIR}/include")

file (GLOB SRCS_MAIN "src/*.cpp")
file (GLOB HDRS_MAIN "include/*.h")
file (GLOB DEFS_MAIN "src/*.def")

add_library (my_lib SHARED ${SRCS_MAIN} ${HDRS_MAIN} ${DEFS_MAIN})

# Uncomment the following line to set unicode encoding as the default.
#target_compile_definitions (my_lib PRIVATE -D_UNICODE -DUNICODE)

# test_project
add_subdirectory ("${PROJECT_SOURCE_DIR}/test_project")

set_property (DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT test_project)
```

- `add_library (my_lib SHARED ${SRCS_MAIN} ${HDRS_MAIN} ${DEFS_MAIN})`
   Add a new library in this solution, the type is SHARED which means it is a dll library. In Vistual Studio, each library is created as a project in solution. The library name is **my_lib** here.

- `add_subdirectory ("${PROJECT_SOURCE_DIR}/test_project")`
   Add a sub-dir, in which contains another **CMakeLists.txt**. Here, the sub-dir is the main dir of test project which is used to test the dll as I mentioned before.

- `set_property (DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT test_project)`
   Set the startup project. Each Vistual Studio solution has one startup project which is used as the entry point of the whole solution.

## The CMakeLists.txt of test project

```cmake
include_directories ("${PROJECT_SOURCE_DIR}/include")

file (GLOB SRCS_EXPL "*.cpp")
file (GLOB HDRS_EXPL "*.h")

add_executable (test_project ${SRCS_EXPL} ${HDRS_EXPL})
target_link_libraries (test_project my_lib)
```

The test project is a seperate from the dll project. So it has it's own file hierarchies and cmake build system except the invoking of dll library.

This **CMakeLists.txt** is pretty simple.

- `add_executable (test_project ${SRCS_EXPL} ${HDRS_EXPL})`

   Generate executable file of Windows, i.e. exe file, instead of dll file.

- `target_link_libraries (test_project my_lib)`

   To compile and run this test project, the **my_lib** library is needed and will be linked in.

## Build and run

Build this solution is simple. Make a tempery directory named **build** in the root of solution, run `cmake ..` in the **build** direcory. Doing this will generate Visual Studio solution files in the build directory. Open this new VS solution in Vistual Studio.

---

### ¶ The end
