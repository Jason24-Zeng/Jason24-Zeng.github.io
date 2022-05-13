---
title: gdb debug 调试
date: 2022-03-19 17:04:15
tags:
- gdb
- debug
category:
- debug tools
password: 0418
cover: /img/gdb-debug-调试/gdb.jpeg
top_img: /img/gdb-debug-调试/gdb.jpeg
---

### Reference

[GDB是什么？](http://c.biancheng.net/view/8123.html)

[linux - gdb cannot attach to process - Stack Overflow](https://stackoverflow.com/questions/45171339/gdb-cannot-attach-to-process)

[gdb调试运行中的进程报错:Operation not permitted. - 代码先锋网](https://www.codeleading.com/article/57791989159/)

[在 mac 下安装 GNU 软件包](https://segmentfault.com/a/1190000011755622)

[PermissionsDarwin](https://sourceware.org/gdb/wiki/PermissionsDarwin)

### 前言

GDB 是 GNU Debugger 的缩写，是 UNIX 及 UNIX-like 系统的强大调试工具。可以用于对 c，c++, fortan, go，java 等的调试。是一个程序员必须装备的利器。本篇文章主要以 c++ 程序为例，介绍 GDB 启动调试的多个方法。

语法错误可以通过编译器诊断，但是逻辑错误无法通过编译器解决。而在实际场景中解决逻辑错误最高效的方法，就是借助调试工具对程序进行调试。而所谓调试，则是给代码设置多个断点，让代码一步一步慢慢执行，跟踪程序在运行过程中的状态。我们可以让程序停在某地，产看当前所有变量的值，或者内存中的数据。

通过调试程序，我们可以监控程序执行的每个细节，包括变量的值，函数的调用过程，内存中的数据，线程的调度等。

总的来说，借助 GDB 调试器可以实现以下几个功能：

1. 程序启动时，可以按照我们自定义的要求运行程序，例如设置参数和环境变量。

2. 可使被调试程序在指定代码处暂停运行，并查看当前程序的运行状态，例如变量值，函数执行结果等，支持断点调试。

3. 程序执行过程中，可以改变某个变量的值，甚至可以改变代码的执行顺序，从而尝试修改程序中出现的逻辑错误。

开发人员要想从事 Linux C/C++, 熟练使用 GDB 调试是一项基本要求。

### GDB 调试的对象

对于 C/C++ 而言，我们需要在编译时加上 -g 参数，保留调试信息，否则无法使用 GDB 进行调试，但是如果不是自己编译的程序，我们可以如何判断文件是否带有调试信息呢？

例如当前有执行二进制文件 `test`，我们可以通过以下方法测试

#### `gdb` 文件方法

执行 

```shell
gdb test
```

如果没有调试信息，会提示如下

```shell
Reading symbols from test...(no debugging symbols found)...done.
```

而如果可以进行调试，则不会出现 `no debugging symbols found` 的指令。

```shell
Reading symbols from test...done.
```

#### `readelf` 查看段信息

执行如下指令

```shell
readelf -S test | grep debug
```

如果有如下 debug 信息，则可以被调试，否则，不能被调试。

```shell
  [25] .zdebug_abbrev    PROGBITS         0000000002c59000  02821000
  [26] .zdebug_line      PROGBITS         0000000002c59119  02821119
  [27] .zdebug_frame     PROGBITS         0000000002e4921f  02a1121f
  [28] .zdebug_pubnames  PROGBITS         0000000002ecd2b2  02a952b2
  [29] .zdebug_pubtypes  PROGBITS         0000000002eda48b  02aa248b
  [30] .debug_gdb_script PROGBITS         0000000002f0ca06  02ad4a06
  [31] .zdebug_info      PROGBITS         0000000002f0ca30  02ad4a30
  [32] .zdebug_loc       PROGBITS         000000000327f304  02e47304
  [33] .zdebug_ranges    PROGBITS         00000000034f675f  030be75f
```

#### `file` 查看 strip 状况

通过执行如下指令

```shell
file test
```

如果返回结果最后是 stripped，表示该文件的符号表信息与调试信息已被去除，无法使用 gdb 调试。而 not stripped 的情况则说明能够被调试。

```shell
ss: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, not stripped
```

### 使用调试方式运行程序

#### 无参程序启动调试

启动 gdb，再执行 `run` 即可

```shell
# 启动 gdb
gdb test

# 进入 gdb 子进程
(gdb)

# 运行
(gdb) run
```

#### 带参数程序的启动调试

假设程序如下，需要启动时带上启动参数

```c++
#include<stdio.h>
int main(int argc, char *argv) {
    if (1 >= argc) {
        printf("usage:hello name\n");
        return 0;
    }
    printf("Hello World %s!\n", argv[1]);
    return 0;
}
```

此时编译时要求带上 debug 信息。则编译如下

```shell
gcc -g -o test test.cc
```

这种情况下的调试方法稍有不同。
可以再执行时带上参数，如下

```shell
gdb test
# 进入子进程，并带上参数执行 test binary
(gdb) run Jason
# 返回结果
Starting program: /home/zijian/workspace/cpp/test Jason
Hello World Jason!
[Inferior 1 (process 20084) exited normally]
```

或者直接 set args，在执行 run，如下

```shell
gdb test
# 进入子进程，并首先设置参数
(gdb) set args Jason
# 再执行程序
(gdb) run
# 返回结果
Starting program: /home/zijian/workspace/cpp/test Jason
Hello World Jason!
[Inferior 1 (process 20084) exited normally]
```

也可以直接制定参数再进入子进程

```shell
gdb --args binary arg1 arg2
```

#### 调试 core 文件

当程序发生 core dump 时，可能会产生 core 文件，这个 core 文件一般能很容易得帮我们定位发生问题的原因和位置。不过前提是系统没有限制 core 文件的产生。

首先检查是否有产生 core 文件的 limit 情况

```shell
ulimit -c
```

如果返回为 0，则表示程序不会产生 core dump 文件。如果需要产生 core 文件，可以执行如下指令，修改 limit 

```shell
# 不限制 core 的大小
ulimit -c unlimited
# 设置最大大小，单位是 block，每个 block 包含 512 字节
ulimit -c 10    
```

调试 core 文件的指令也比较简单，指令如下即可进入调试

```shell
gdb binary_path core_path
```

这块后面会具体学习

#### 调试已运行程序

如果这个程序正在运行，我们需要首先获得进程 ID，这时我们可以使用 `ps` 或 `pidof` 两个指令。

```shell
ps -ef | grep 进程名

# 结果大致如下
root@c30ece665722:/data/videosearch-ss/make/output/bin# ps -ef | grep ss
root      3977     1  0 07:58 pts/1    00:02:57 /data/videosearch-ss/make/output/bin//ss -log_path ./logs/
root      4574    17  0 19:44 pts/1    00:00:00 grep --color=auto ss


#或者
pidof 进程名

#这会返回进程名
3977
```

##### 使用 gdb `attach` 进程

假设通过以上方法，我们已经获得了进程名，则我们可以通过下面的方式进行调试

```shell
# 进入 gdb 子程序
gdb
# attach 进程
(gdb) attach 3977
```

这样就可以继续进行调试了。

而执行时可能出现如下错误提示

```shell
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
```

解决方法是切换到 root 用户，然后修改 `/etc/sysctl.d/10-ptrace.conf` 配置中的指令即可。但是个人亲测方法不行，可能是配置没有生效或者什么原因，需要花时间调试一下。

```shell
kernel.yama.ptrace_scope = 0
```

或者在运行 docker 时，使用 `--privileged` 选项

```shell
docker exec --privileged -it <container> /bin/bash
```

而一个更细粒度的方法，是使用 `cap_add`，这要求我们启动容器使用 `CAP_SYS_PTRACE` 容量，使用 `--cap-add=SYS_PTRACE` 指令。

对使用容器引擎的情况，我们需要尝试从容器外部(host) 去 attach to 进程，它可能会有一个不同的 PID。

##### 直接调试相关 id 进程

通过执行 `gdb program pid`，可以直接实现上述方法。

```shell
gdb us 3977
# 或者
gdb us --pid 3977
```

##### 其他问题，运行程序无调试信息

通常为了节省磁盘空间，已经运行的程序通常没有调试信息，在不重启程序的情况下，我们应该怎么做呢？我们可以用同样的代码，再编译一个带调试信息的版本，然后执行 file 和 attach 操作即可。

```shell
gdb
(gdb) file test
(gdb) attach 3977
```

#### `gdb` 内基本操作

进入 gdb 后，有一些基础指令一定需要牢记，对 binary 的整个执行调试过程都非常的重要。

- `refresh` ，或者快捷键 `ctrl + l`，刷新显示页面，有时界面会出现排序问题，可以使用 refresh 刷新

- `run`，运行程序

- `layout next`，查看代码，这时就进入了显示页面

- `break POINT` ，设置断点，可以是行号，函数名等。

- `next`，进入下一个 step，`n`  for short

- `continue`，Continue 到下一个断点

- `print VARIABLE` ，打印某个变量值

- `print *arr@len`，打印一个数组

- `watch VARIABLE` 观察一个变量的变化，也就是当我们进入 gdb 后，设置对某个变量的观察，随着我们 step-by-step 得运行指令或者运行到下一个断点前。当该变量在这个间断中发生变化，都会有改变信息打出来。主要是导致改变的行号以及变化的内容。

- `list` 或 `l`，我们可以查看代码的某一部分。

- `frame` 或`f` 展示代码目前运行到了哪个位置

- `step` 或 `s` 表示进入当前行的函数内部

- `backtrace` 或 `bt`， 则是与 `step` 相辅相成的一个概念，即生成这个函数与 `main` 函数之间的调用关系。

- `info b` 列举所有断点的信息

- `delete BREAK_NUM` 删除某个 `info b` 中对应 `NUM` 的断点

- 

#### 训练前准备

首先我们需要在 mac 上安装 Homebrew 并用它来安装相关 GNU 编译工具，以及 gdb 等插件。

```shell
# 安装 Homebrew

# 安装 bash
brew install bash

# 安装 coreutils
brew install coreutils

# 安装 gawk
brew install gawk

# 安装 gnu-sed
brew install gnu-sed

# 安装 gdb，并顺带安装 tmux
brew install gdb

# 修改 ~/.bashrc，~/.vimrc 与 ~/.dir_colors 使 base 的可视化效果更好
vim ~/.bashrc
export PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"
alias ls='ls -F --show-control-chars --color=auto'
eval `gdircolors -b $HOME/.dir_colors`
alias awk=gawk
alias sed=ased

vim ~/.vimrc
syntax on

gdircolors --print-database > ~/.dir_colors
```

##### 
