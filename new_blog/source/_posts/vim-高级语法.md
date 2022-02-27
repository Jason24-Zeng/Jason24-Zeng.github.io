---
title: command line 高级语法
date: 2022-02-19 17:54:34
tags: 
- vim
- cmd
categories:
- tools
- vim
- cmd line
cover: /img/command-line/1200px-Vimlogo.svg.png
top_img: /img/ElasticSearch/1200px-Vimlogo.svg.png
---

## vim 高级操作

### 输出清晰表格

命令行参数输出为

```shell
column -t
```

其实就是以 `\t` 作为分隔符将每行的数据分成不同列
例子：

```
mount | column -t
```

相应的指令

| -s sep     | 指定一组字符，用于分隔-t选项的列    |
| ---------- | -------------------- |
| -t         | 确定输入包含的列数并创建表        |
| -n         | 禁用将多个相邻分隔符合并为单个分隔符行为 |
| -c columns | 输出格式化为显示columns列宽    |
| -x         | 在填充行之前填充列            |
| -e         | 不要忽略空白行              |

### 重复执行某段代码

在命令行或者 shell 脚本中经常使用的 while 循环就可以完成这样的操作

```shell
while true
do 
ping -c 1 google.com > /dev/null 2>&1 && break
done ;
```

其中还牵涉到了重定向 `> /dev/null 2>&1` 的操作。

#### 重定向

| 命令              | 说明                             |
| --------------- | ------------------------------ |
| command > file  | 将输出重定向到 file。                  |
| command < file  | 将输入重定向到 file。                  |
| command >> file | 将输出以追加的方式重定向到 file。            |
| n > file        | 将文件描述符为 n 的文件重定向到 file。        |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。  |
| n >& m          | 将输出文件 m 和 n 合并。                |
| n <& m          | 将输入文件 m 和 n 合并。                |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

重定向符 `>` 与 `<` 等的操作如上。另外需要注意的是

> 文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。

### 对进程进行排序

 按照内存资源进行排序 

```shell
ps aux | sort -nk 4
```

按照 CPU 资源的使用量对进城进行排序

```shell
ps aux | grep -nk 3
```

### 返回之前的目录

```shell
cd -
```

### 会话关掉之后继续运行环境

```shell
nohup ping -c 10 google.com &
```

 **nohup** 是一个 [POSIX](https://zh.wikipedia.org/wiki/%E5%8F%AF%E7%A7%BB%E6%A4%8D%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%8E%A5%E5%8F%A3 "可移植操作系统接口") 命令，用于忽略 `SIGHUP` ("**sig**nal **h**ang **up**" 译：挂断信号) 。 `SIGHUP`信号是终端注销时所发送至程序的一个信号。

最后的 `&` 则是表示后台执行的意思

### 创建指定大小的文件

```shell
dd if=/dev/zero of=out.txt bs=1M count=10
```

### 以 root 用户来执行最后一个指令

当我们发现用一般用户无法运行指令时，我们可以切换到 root 用户去执行，而使用一下指令，可以直接使用 root 用户执行操作前的最后一个指令

```shell
sudo !!
```

### `tr` 命令

我们可以用 `tr` 命令替换任意字符

#### 标签符号替换空格符

```shell
cat text.txt | tr `:[space:]` ` ` > out.txt
```

#### 大写转小写

```shell
cat myfile | tr a-z A-Z> output.txt
```

### Xargs 指令

我们可以使用这个命令将命令的输出作为参数传递给另一个命令，比如

```shell
find. -name *.png -type f -print | xargs tar -cvzf images.tar.gz
cat urls.txt | xargs wget
ls /etc/*.conf | xargs -i cp {} /home/likegeeks/Desktop/out
```

其中，第三行的 `-i` 与 `{}` 制定了命令行输出的位置

## vim 使用操作

### 窗口管理

- `:sp`，split，水平划分。前面加上数字以设置新的窗口高度

- `:vs`, vertical split，垂直划分。

- `ctrl + w + w` 光标切换到下一个窗口

- `ctrl + w + h/j/k/l` 将光标沿方向更改到指定窗口

- `ctrl + w + c` 关闭当前窗口

- `ctrl + w+` 增大窗口大小

- `ctrl + w-` 减少窗口大小

- `:only` 关闭当前窗口以外的所有窗口

- `ctrl + w + n` 打开新窗口，窗口使用新的缓冲区。

### Tab 管理

我们可以分别管理每个选项卡的窗口布局。要创建标签，我们可以使用`:tabnew`命令打开一个新标签。

一些简单的选项卡管理方法是:

- `:tabnew`:打开新标签
- `:tabclose`:关闭当前标签页
- `:tabn`:切换到下一个标签（**n**ext）
- `gt`:切换到下一个标签
- `:tabp`:切换到上一个标签
- `gT`:切换到上一个标签
- `:tab ball`:在单个选项卡中打开所有缓冲区
- `:tabs`:列出所有可用的标签

随着缓冲区，窗口和选项卡的混排，有时会混淆您当前正在查看的文件。查找当前正在查看的文件名的快速方法是键入:

- **CTRL-g**:显示当前文件名

### `vimrc` 的使用

在 `vim` 中，我们可以通过 `:version` 命令查看 vim 载入配置的优先顺序，比如我所在的 macOS 系统的返回结果为：

```shell
   system vimrc file: "$VIM/vimrc"
     user vimrc file: "$HOME/.vimrc"
 2nd user vimrc file: "~/.vim/vimrc"
      user exrc file: "$HOME/.exrc"
       defaults file: "$VIMRUNTIME/defaults.vim"
  fall-back for $VIM: "/usr/share/vim"
```

可以看到 vim 会有限读取 user vimrc file: `"$HOME/.vimrc"`，当这个文件不存在时，vim 会去寻找 2nd user vimrc file：`"~/.vim/vimrc"`。了解这个对以后查询 `vim` 不生效的原因很重要。

### 自动补全

#### 单词补全

这种补全方式属于单词的前缀匹配，在 Insert 模式下，我们输入一些单词，然后按 `ctrl + n`， vim 会自动出现下拉菜单，且默认选中第一个单词，此时可以使用上下光标进行单词的选择。而 `ctrl + p` 的功能也是这样的，只是默认选中列表的最后一个单词。

#### 行补全

这种补全方式不再只补全其中一个单词，而是自动补全整句，使用的操作命令顺序为：`ctrl + x` 、 `ctrl + l`

#### 字典补全

假定有一个候选字典表 `my_diction.txt`，里面每一行都会有一个单词，我们可以载入这个词典表，基于这个词典表进行 Vim 自动补全，设置步骤为：

1. 在 `$HOME/.vimrc` 配置文件中加入: `set dictionary-=~/dict.txt dictionary+=~/dict.txt`

2. 打开 `vim`，在插入模式下输入 `ctrl + x` 后输入 `ctrl + k`，就可以匹配字典中的单词

3. 如果想要使用单词补全的方法显示列表，即使用 `ctrl + n`，可以考虑配置 `.vimrc` 文件，加入: `set complete-=k complete+=k`

### Reference

[15个超实用的Linux 命令行使用技巧](https://zhuanlan.zhihu.com/p/47383299)

[Vim 高级使用技巧汇总](https://zhuanlan.zhihu.com/p/90721457)

[Beginners Guide to Tabs in Vim](https://webdevetc.com/blog/tabs-in-vim/)

https://www.youtube.com/watch?v=J0-N1nVTU4k

[Vim 从入门到精通](https://github.com/wsdjeg/vim-galore-zh_cn#%E4%BB%80%E4%B9%88%E6%98%AF-vim)

[Vim自带自动补齐功能-Vim入门教程(11)](https://vimjc.com/vim-auto-complement.html)
