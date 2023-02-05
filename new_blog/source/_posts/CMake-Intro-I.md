---
title: Cmake-Intro-I -- CMakeLists.txt
date: 2022-08-03 22:07:41
tags: 
- C++
- CMake
categories:
- Build
password: 41831
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

### 三、指定项目名及版本等

```cmake
project(us)
set(VERSION "0.0.1")
set(CPACK_PACKAGE_RELEASE 1)
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
# 0x01. CPM 安装
include(FetchContent)
if(CPM_SOURCE_PATH)
    get_filename_component(CPM_SOURCE_PATH ${CPM_SOURCE_PATH} ABSOLUTE)
    set(CPM_DOWNLOAD_LOCATION "${CPM_SOURCE_PATH}/cpm/CPM.cmake")
elseif(DEFINED ENV{CPM_SOURCE_PATH})
    set(CPM_DOWNLOAD_LOCATION "$ENV{CPM_SOURCE_PATH}/cpm/CPM.cmake")
else()
    set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM.cmake")
endif()

if(NOT (EXISTS ${CPM_DOWNLOAD_LOCATION}))
  message(STATUS "Downloading CPM.cmake to ${CPM_DOWNLOAD_LOCATION}")
  FetchContent_Declare(
      CPM
      GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpm.cmake
      GIT_TAG master
      SOURCE_DIR ${CMAKE_BINARY_DIR}/cmake/CPM
  )
  FetchContent_MakeAvailable(CPM)
  file(CREATE_LINK
       ${CMAKE_BINARY_DIR}/cmake/CPM/cmake/CPM.cmake
       ${CPM_DOWNLOAD_LOCATION}
       SYMBOLIC
  )
endif()

include(${CPM_DOWNLOAD_LOCATION})


# BUILD_TYPE: Debug Release RelWithDebInfo MinSizeRel
# Set a default build type for single-configuration
# CMake generators if no build type is set.
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
message(STATUS "CMAKE_BUILD_TYPE of ${PROJECT_NAME} = ${CMAKE_BUILD_TYPE}")


# 0x02. git 相关配置
# get current git version
execute_process(
    COMMAND git describe --abbrev=12 --dirty --always --tags
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_VERSION
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# get commit date
execute_process(
    COMMAND git show -s --format=%cd --date=iso
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_COMMIT_DATE
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# get commit author
execute_process(
    COMMAND git show -s --format=format:%an
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_COMMIT_AUTHOR OUTPUT_STRIP_TRAILING_WHITESPACE
) 

# 0x03. gflags
unset(GFLAGS_INCLUDE_PATH CACHE)
find_path(GFLAGS_INCLUDE_PATH NAMES gflags/gflags.h NO_CACHE)
if(NOT GFLAGS_INCLUDE_PATH)
    execute_process(COMMAND apt-get install -y libgflags-dev)
endif()

# 0x04. snappy
unset(SNAPPY_INCLUDE_PATH CACHE)
find_path(SNAPPY_INCLUDE_PATH snappy.h)
if(NOT SNAPPY_INCLUDE_PATH)
    execute_process(COMMAND apt-get install -y libsnappy-dev)
endif()

# 0x05. leveldb
unset(LEVELDB_INCLUDE_PATH CACHE)
find_path(LEVELDB_INCLUDE_PATH leveldb/db.h)
if(NOT LEVELDB_INCLUDE_PATH)
    execute_process(COMMAND apt-get install -y libleveldb-dev)
endif()


# 0x05. set depends
CPMAddPackage(
    NAME fmt
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/third-party/fmt.git
    GIT_TAG master
)

CPMAddPackage(
    NAME fasttext
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/third-party/fasttext.git
    GIT_TAG master
)

CPMAddPackage(
    NAME rapidjson
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/third-src/rapidjson.git
    GIT_TAG master
)

CPMAddPackage(
    NAME ulbase
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpputil/ulbase.git
    GIT_TAG master
)

CPMAddPackage(
    NAME ullog
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpputil/ullog.git
    GIT_TAG master
)

CPMAddPackage(
    NAME design_pattern
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpputil/design_pattern.git
    GIT_TAG master
)


CPMAddPackage(
    NAME dyn_resource
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpputil/dyn_resource.git
    GIT_TAG master
)

CPMAddPackage(
    NAME uldict
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpputil/uldict.git
    GIT_TAG v1.0.6
)

CPMAddPackage(
    NAME json_enhanced
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpputil/json_enhanced.git
    GIT_TAG master
)

CPMAddPackage(
    NAME design_pattern
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/cpputil/design_pattern.git
    GIT_TAG master
)

CPMAddPackage(
    NAME snutil
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/online/snutil.git
    GIT_TAG master
)

CPMAddPackage(
    NAME daproto
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/online/daproto.git
    GIT_TAG master 
)

CPMAddPackage(
    NAME event-tracking-proto
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/online/event-tracking-proto.git
    GIT_TAG master
)

CPMAddPackage(
    NAME usproto
    GIT_REPOSITORY https://git.garena.com/shopee/ai-engine-platform/video_home_search/online/usproto.git
    GIT_TAG master
)
```

### 七、设置编译参数

```cmake
set(GIT_URL "http://git.garena.com:shopee/ai-engine-platform/video_home_search/online/us")

# CMAKE_CXX_FLAGS is applied to both compiler and linker
set(CMAKE_CXX_FLAGS "-Wall -Werror -fPIC")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D__FILENAME__='\"$(notdir $<)\"'")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DGIT_VERSION='\"${GIT_VERSION}\"'")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCOMMIT_DATE='\"${GIT_COMMIT_DATE}\"'")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCOMMIT_AUTHOR='\"${GIT_COMMIT_AUTHOR}\"'")

set(CMAKE_CXX_FLAGS_RELEASE "-O3")
set(CMAKE_CXX_FLAGS_DEBUG  "-O0 -g")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -g")

#  add_definitions is applied only to compiler.
add_definitions("-Wno-unused-function -Wno-unused-variable -Wno-sign-compare -fno-omit-frame-pointer")
add_definitions("-Wno-class-memaccess -Wno-unused-value -Wno-unused-but-set-variable")
```

### 八、include 路径

```cmake
include_directories("${PROJECT_SOURCE_DIR}")
include_directories("${PROJECT_SOURCE_DIR}/src")
inlucde_directories("${PROJECT_BINARY_DIR}")
```

### 九、设置一些 third-party 参数

```shell
set(WORKROOT ./) 
#set(THIRD_PARTY ${WORKROOT}/third-party)
#set(SEARCH ${WORKROOT}/online)
set(SEARCH ./)
get_filename_component(SEARCH "./" ABSOLUTE)
MESSAGE( STATUS "SEARCH ABS_PATH is key = ${SEARCH}")

set(THIRD_PARTY /third-party)
#set(PROTOBUF ${THIRD_PARTY}/protobuf)
set(LIBCONFIG ${THIRD_PARTY}/libconfig)
set(RE2 ${THIRD_PARTY}/re2)
set(BRPC ${THIRD_PARTY}/brpc-with-naming)
set(ZOOKEEPER ${THIRD_PARTY}/zookeeper)
set(GPERFTOOLS ${THIRD_PARTY}/gperftools)
set(NAMING ${THIRD_PARTY}/naming)
```

### 十、add_subdirectory

```shell
add_subdirectory(src/core)
add_subdirectory(public)
add_subdirectory(strategy)
add_subdirectory(util)
add_subdirectory(frame)
add_subdirectory(model/features)
add_subdirectory(model/video_user_recall)
add_subdirectory(model/nlp_analysis)

add_subdirectory(business/livestreaming/model/query_category_predict)
add_subdirectory(business/livestreaming/strategy)
add_subdirectory(business/livestreaming/startup)
add_subdirectory(business/video/startup)
add_subdirectory(business/tti2i/strategy)

add_subdirectory(model/query_topic_cls)
add_subdirectory(model/query_search_intent)

add_subdirectory(src/clients)
add_subdirectory(src/search/cmd_imp)
add_subdirectory(src/search/base)
add_subdirectory(src/strategy)
```

### 十一、安装其他

```shell
string(REGEX MATCH "^/.*" TARGET_MATCHED ${OPT_INSTALL_TARGET})
message(STATUS "TARGET_MATCHED=${TARGET_MATCHED}")
if(NOT TARGET_MATCHED)
    set(OPT_INSTALL_TARGET ${PROJECT_SOURCE_DIR}/${OPT_INSTALL_TARGET})
endif()
set(CMAKE_INSTALL_PREFIX ${OPT_INSTALL_TARGET})
message(STATUS "Install target to ${CMAKE_INSTALL_PREFIX}")
install(TARGETS us
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib.so
    ARCHIVE DESTINATION lib)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/bin/ DESTINATION bin
    FILE_PERMISSIONS OWNER_EXECUTE GROUP_EXECUTE)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/conf/ DESTINATION conf)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/values/ DESTINATION values)
install(CODE "execute_process(COMMAND ${PROJECT_SOURCE_DIR}/bin/generate_conf.sh --dir_base=${CMAKE_INSTALL_PREFIX})")

if(OPT_INSTALL_DATA_WITH_SYMBLINK)
    set(DATA_INSTALL_MODE "symb")
else()
    set(DATA_INSTALL_MODE "copy")
endif()
install(CODE "execute_process(COMMAND ${PROJECT_SOURCE_DIR}/bin/install_data.sh
                                -i ${PROJECT_SOURCE_DIR}/data
                                -o ${CMAKE_INSTALL_PREFIX}/data
                                -m ${DATA_INSTALL_MODE})")
```

## set 使用讲解

在 set 一些参数时，我们可能会碰到一些一眼无法识别其含义的超参数，比如

```cmake
set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "build type")
```

中出现的 `CACHE` and `STRING` 的含义。这个 Section 主要就是尝试解释这部分含义。

可以参考 [Cmake set Intro](https://cmake.org/cmake/help/v3.0/command/set.html)，

首先，什么是 CACHE，中文翻译是缓存，即暂时存放到某一个"存储"中，这样当多次调用的时候，都可以从这个"存储"中获取，获取的速度会比较快。在 cmake 中，需要区分两个概念：normal variable 和 cache variable，普通变量和缓存变量。普通变量是一个局部变量，只能在同一个工程中使用，会有作用域限制和区分。缓存变量是一个全局变量，在同一个CMake工程中任何地方都可以使用。````

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
```

> Within CMake sets `<variable>` to the value `<value>`. `<value>` is expanded before `<variable>` is set to it. Normally, set will set a regular CMake variable. If CACHE is present, then the `<variable>` is put in the cache instead, unless it is already in the cache. See section ‘Variable types in CMake’ below for details of regular and cache variables and their interactions. If CACHE is used, `<type>` and `<docstring>` are required. `<type>` is used by the CMake GUI to choose a widget with which the user sets a value.

通常，set 会设置一个常规的 Cmake 变量，如果 CACHE 这个 term 存在，则 变量会被放到 cache 里，除非它已经在 cache 中了。当 使用 CACHE 时，类型和文本字符串必须被提供，类型被 CMake GUI 使用去选择哪个 widget 用户设定一个值。

**还有一种方法能够设置CACHE变量，就是通过cmake命令的-D选项，可以添加一个CACHE变量**

CACHE作用如下：

- 如果缓存中存在同名的变量，根据FORCE来决定是否写入缓存：如果没有FORCE，这条语句不起作用，使用缓存中的变量；如果有FORCE，使用当前设置的值。
  
  - 注意，如果是FORCE，也能修改-D选项设置的CACHE变量，所以有可能传入的生成命令选项是无效的。

- 如果缓存中不存在同名的变量，则将这个变量写入缓存并使用。

缓存变量也可以设置只在本文件内生效，将STRING类型改为INTERNAL即可。

### Cache 总结

正常使用的时候，如果有多层CMakeLists.txt，需要跨文本的变量，应该使用CACHE类型，如果只是当前文本的变量，则不需要使用CACHE，更重要的是，应该避免使用同名的普通和缓存变量。另外，由于CMake没有有效的清除缓存的方法，如果要彻底清除缓存，需要删除build或者release文件夹的所有文件。

### OpenSSL 安装 mac

```shell
brew update
brew install openssl
echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```



## Reference

[如何用cmake编译](https://zhuanlan.zhihu.com/p/59161370#:~:text=CMake%E6%98%AF%E4%B8%80%E7%A7%8D%E8%B7%A8,so(shared%20object)%EF%BC%89%E3%80%82)

[CMake中变量总结 | 拾荒志](https://murphypei.github.io/blog/2018/10/cmake-variable)
