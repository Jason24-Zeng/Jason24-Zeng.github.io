---
title: git 子模块
date: 2022-03-02 22:58:07
tags: 
- git 
- Devops
category:
- cmd line tools
- git
cover: /img/git-多人开发-与-分支管理/git-flow.webp
top_img: /img/git-多人开发-与-分支管理/git-flow.webp
---

### 前言

我们经常碰到这样的情况，当我们在开发某一个项目时，我们需要引入另一个项目的某一部分，它可能是一个第三方库，或者是我们自己开发的一个可复用的项目。这时我们可能希望能区分开这两个项目，但是能在其中一个项目中引用另一个项目。

我们的一种可能解决方法，是将其中一个项目(比如 library)完整得 copy 到自己的项目树里。但是可能这会导致个性化这个 library 很困难，而且得确保每个客户端都能包含该库 。另外， 如果将代码复制到自己的项目中，那么你做的任何自定义修改都会使合并上游的改动变得困难。

为了解决这个麻烦， git 使用了 submodule，它允许你将一个 git 仓库作为另一个 git 目录的子目录。这样，我们就可以将另一个仓库 clone 到我们自己的项目中，并且保证整个 commits 是分离开的。

### 添加子模块

通过以下指令，我们可以添加子模块（比如 DbConnector）到某一个项目中, 默认情况会放到统计目录中。

```shell
git submodule add https://github.com/chaconinc/DbConnector ## 相对/绝对 url
```

这时会生成两个文件，其中一个是 `DbConnector`，另一个是 `.gitmodules`

#### `.gitmodules` 文件

该文件是用于保存项目 URL 与已经拉取的本地目录之间的映射

```shell
[submodule "DbConnector"]
    path = DbConnector
    url = https://github.com/chaconinc/DbConnector
```

多个子模块则会有多条记录，该文件也想 `.gitignore` 文件一样通过版本进行控制。它会和项目的其他部分一同被拉取推送。

#### 子目录文件

虽然 `DbConnector` 是工作目录中的一个子目录，但 Git 还是会将它视作一个子模块。当你不在那个目录中时，Git 并不会跟踪它的内容， 而是将它看作子模块仓库中的某个具体的提交。

当你提交时，会看到类似下面的信息：

```console
$ git commit -am 'added DbConnector module'
[master fb9093c] added DbConnector module
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 DbConnector
```

注意 `DbConnector` 记录的 `160000` 模式。 这是 Git 中的一种特殊模式，它本质上意味着你是将一次提交记作一项目录记录的，而非将它记录成一个子目录或者一个文件。

最后，推送这些更改：

```console
$ git push origin master
```

### clone 含有子模块的项目

当我们 clone 某一个含有子模块的项目时，如下：

```console
 git clone https://github.com/chaconinc/MainProject
```

我们会发现子模块目录下是空的，这时候我们还没有 clone 相应的子模块，通过执行以下两个指令，我们通过`git submodule init` 用来初始化本地配置文件，并使用 `git submodule update` 则从该项目中抓取所有数据并检出父项目中列出的合适的提交。

另外，还可以通过以下指令，一步完成 clone 操作，甚至完成嵌套子模块的 clone

```shell
git clone --recurse-submodules https://github.com/chaconinc/MainProject
```

### 待续

### Reference

[Git - 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
