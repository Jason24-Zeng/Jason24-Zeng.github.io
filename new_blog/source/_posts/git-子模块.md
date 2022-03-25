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

### clone 但忘记拉取子模块

如果我们已经将项目 clone 到了本地，但是忘记拉取子模块，则我们可以执行如下指令：

```shell
git submodule update --init
```

它会执行 `git submodule init` 与 `git submodule update` 指令，去拉取并检出子模块。

另外，如果子模块里面还有嵌套，我们可以执行以下指令，完成嵌套式的拉取和检出流程：

```shell
git submodule update --init --recursive
```

### 拉取子模块远端最新代码

我们可能不会修改子模块代码，但是想要接受其子模块的最新修改，则我们可以进入子模块目录，执行 `git fetch` 和 `git merge` 指令，合并上游分支的新代码到本地。并使用 `git diff --submodule` 可以看到子模块修改的内容。如果我们这时提交更新，会让子模块锁定为别人修改后的新代码。即大家 `git submodule update --init` 初始化子模块代码会变成修改后的子模块代码。

除此以外，还有一个更方便的指令完成上述拉取并检出子模块新修改的方法。就是使用如下指令

```shell
git submodule update --remote <submodule>
```

### 拉取主模块代码

注意拉取主模块代码只执行 `git pull` 或者 `git pull --rebase` 是不够的，它只是检查到了子模块有修改，并递归得抓取修改，但是它并不会更新子模块，想要更新子模块，我们还需要执行 `git submodule update --init --recursive`。或者 `git pull` 加上参数 `--recurse-submodules` 实现自动化。

在父级项目中，可能出现这样的情况：当我们拉取的提交中， 可能 `.gitmodules` 文件中记录的子模块的 URL 发生了改变。这时候执行 `git submodule update` 或者`git pull --recurse-submodules` 可能会发生错误，我们需要将新的 URL 复制到本地中:

```shell
git submodule sync --recursive
```

再使用新的 URL 更新子模块即可。

### Reference

[Git - 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
