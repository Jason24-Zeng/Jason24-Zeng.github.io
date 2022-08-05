---
title: Cake-Intro-I
date: 2022-08-03 22:07:41
tags: 
- C++
- CMake
categories:
- BuildTools
---



## 前言

Cmake 是一种适用于大型项目的 C++ 跨平台编译工具，通过提前设定 CMakeLists.txt 文件，使用 cmake 指令生成 make 所需 makefile，从而生成指导编译的向量图，再根据向量图依次编译生成相关依赖和 .o 文件，最终链接生成 binary。因为工作得需要，我需要习惯 Cmake 编译的整体流程和简单语句，因此写下这篇博客，帮助理解 和 remind Cmake 的使用。



## Cmake 下载 - Install on CentOs

执行

```shell
sudo yum install cmake gcc-c++ make
```

去安装 cmake gcc 以及 make



## 编写 `CMakeLists.txt` 文件

### 一、设定 `cmake` 最低版本要求

```cmake
cmake_minimum_required(VERSION 3.6)
```

### 二、指定 C 与 C++ 编译器

```cmake
set(CMAKE_C_COMPILER /usr/bin/gcc)
set(CMAKE_CXX_COMPILER /usr/bin/g++)
```

### 三、指定项目名

```cmake
project(us)
```

### 四、specify C++ 标准

```cmake
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED true)
```

### 五、设置一些 options

```cmake
option(OPT_ENABLE_DEBUG_LOG "enable debug log, debug log will disabled by default..", OFF)
option(OPT_INSTALL_DATA_WITH_SYMBLINK "indall data with symblink, enable by default" ON)
set(OPT_INSTALL_TARGET "output" CACHE STRING "the path of dir 'output', 'output' by default")
```

### 六、安装 CPM 等依赖

```cmake
include(FetchContent)
if(CPM_SOURCE_PATH)
    get_filename_component(CPM_SOURCE_PATH ${CPM_SOURCE_PATH} ABSOLUTE)
    set(CPM_DOWNLOAD_LOCATION "${CPM_SOURCE_PATH}/cpm/CPM.cmake")
elseif(DEFINED ENV{CPM_SOURCE_PATH})
    set(CPM_DOWNLOAD_LOCATION "$ENV{CPM_SOURCE_PATH}/cpm/CPM.cmake")
else()
    set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM.cmake")
endif()

... 未完待续
    
```

### 七、设置编译参数

```cmake
## set(GIT_UTL "http://git.xxx/us")

# CMAKE_CXX_FLAGS is applied to both complier and linker
set(CMAKE_CXX_FLAGS "-Wall -Werror -fPIC")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_FILENAME__='\"${notdir $<}\"'")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DGIT_VERSION='\"${GIT_VERSION}\"'")

...未完待续


set(CMAKE_CXX_FLAGS_RELEASE "-O3")
set(CMAKE_CXX_FLAGS_DEBUG  "-O0 -g")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO  "-O3 -g")

```

### 八、include 路径

```cmake
include_directories("${PROJECT_SOURCE_DIR}")
include_directories("${PROJECT_SOURCE_DIR}/src")
inlucde_directories("${PROJECT_BINARY_DIR}")
```







## Reference

[如何用cmake编译](https://zhuanlan.zhihu.com/p/59161370#:~:text=CMake%E6%98%AF%E4%B8%80%E7%A7%8D%E8%B7%A8,so(shared%20object)%EF%BC%89%E3%80%82)
