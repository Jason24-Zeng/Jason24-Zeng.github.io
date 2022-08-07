---
title: CMake-Into-II
date: 2022-08-07 13:23:16
tags:-
- shell
- cmake
categories:
- 程序员基础
password: 0418
---





## 执行编译指令 shell 脚本

```shell
readonly CURR_WORK_DIR=$(dirname $(readlink -f $0))
readonly SCRIPT_NAME=${0##*/}
```

Note:

### 1. `readlink`

`readlink` 是 Linux 系统的常用工具，主要用来找出符号连接所指向的位置。官方文档中的解释是

```textile
print valuee of a symbolic link or canonical file name
```

也就是说，它会去得到符号链接的原路径。比如：

```shell
$ readlink /usr/bin/awk
/etc/alternatives/awk ---获取软链原路径，可能还不是真实路径
$ readlink /etc/alternatives/awk
/usr/bin/gawk    --- binary 的真实路径
```

需要注意 `-f` 选项可以理解为 recursively readlink，直到找到符号链接的最原始路径。

```shell
$ readlink -f /usr/bin/awk
/usr/bin/gawk
```

### 2. `$0` and `${0##*/}`

`$0`会展开 shell 或这 shell 脚本的名字，这会在 shell 初始化的时候实现。

如果 Bash 被文件 command invoke， `$0` 被设定为文件名

如果 Bash 启动是使用 `-c` 选项，则 `$0` 被设定为那个字符串执行时的第一个入参，前提是有一个参数存在。

比如：

如果你执行

```shell
/usr/bin/example.sh #execute the shell script
```

`$0` 将会是 `/usr/bin/example.sh`

而如果你当前目录是 `/usr`，你也可以使用下面语句调用同一个脚本

```shell
./bin/example.sh
```

`$0` 将会是 `./bin/example.sh`

`${0##*/}` 和 `${0%/*}`，被称为参数扩展 parameter expansion。

`#`，意味着我们可以展开 `$0` 在去除前面的特定前缀之后，而我们这里的情况则是 `*/` ，如果只有一个 `#`，意味着寻找是非贪婪 (non-greedy) 的，它会找到第一个 `/` 就停止，并去除掉 第一个 `/` 之前的所有内容，而如果有两个 `#`，则它会贪婪得去掉所有 `/` 之前的内容。比如：

```shell
$ var = "foo/bar/baz"
$ echo "${var#*/}"
bar/baz
$ echo "${var##*/}"
baz
```

`${0%/*}` 展开包含在 argument 0 之内的值，在取出 `/*` 的字符串后缀之后，比如

```shell
$ var = "foo/bar/baz"
$ echo "${var%/*}"
foo/bar
```
