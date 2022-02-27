---
title: git 多人开发 与 分支管理
date: 2022-02-27 11:09:29
tags: 
- git
- Devops
category:
- git
cover: /img/git-多人开发-与-分支管理/git-flow.webp
top_img: /img/git-多人开发-与-分支管理/git-flow.webp
---

## git 多人开发

在现实工作中，我们经常会碰到多人开发同一个模块或项目的情况，这中间需要注意许多问题，比如分支的理解，冲突的解决等，这片文章就主要讲讲在用 git 开发代码时可能碰到的问题与解决方法。

### Notice

第一次开发的时候，有开发经验的老司机都会发出的警告：**永远不要在本地 master 分支上开发**，要保留一个干净的分支，以便后续切分支。后来在日常工作中，深刻得体会到了干净分支的重要性。

每个分支只 focus 在一个项目上，不要再同一个分支中搞多项目开发，这会给开发和维护带来很大的困难。

### 抓取分支

在确保本地已经安装 git 的前提下，要想在本地进行 git 项目的开发，首先我们需要让本地机器有这个项目的权限。则需要做两件事：

1. 确定自己有 git 账户对这个项目有开发权限，否则需要申请开发权限。如果没有开发权限，我们就无法自行得让本地连接到项目的远程分支仓库中

2. 本地生成 ssh 秘钥，将公钥 Add 到项目 repo 的可识别 Key 中，这保证了本地机器能通过 ssh 对分支进行开发推送等。还需要注意本地 ssh 可能会有多个密钥，要确保 git 使用的是 post 到项目 repo 中的 key

完成上述两步，应该就可以开始抓取远程分支进行开发了，使用的 cmd line 为

```shell
git clone git@github.com:Jason24-Zeng/Jason24-Zeng.github.io.git $name
```

这个指令的意思就是将远程分支拉取到本地，并写到当前路径下的 `$name` 目录，如果没有就创建。

### 查看 Git 远程库信息

进入 `$name` 的目录，可以通过输入 `git remote -v` 显示详细的远程库信息

```textile
origin    git@github.com:Jason24-Zeng/Jason24-Zeng.github.io.git (fetch)
origin    git@github.com:Jason24-Zeng/Jason24-Zeng.github.io.git (push)
```

origin 是远程仓库的默认名称，fetch 和 push 显示当前具有抓取与推送到 `origin` 的权限。

### 推送分支

当我们本地进行提交后，我们可以将本地提交推送到远程库中。推送时的指令如下

```shell
git push origin master
```

表示将当前本地提交的内容推送到 origin 这个远程仓库的 master 远程分支上。如果推送的结果与前面的结果有冲突，可以先 `git pull` 将当前的远程分支拉到本地，然后将冲突解决后，再 push 到远程分支上。如果远程分支只有一个人使用，且确认希望就将当前的内容完全 push 到远程分支上，则可以使用 `git push -f origin master`，它会清空 远程 `master` 分支的内容，然后把本地分支的内容全部推送给  master。

### 不要在本地 master 分支上开发

再次提到，不要再本地 master 分支上开发，每天让本地 master 分支追踪远程 master 分支，并拉取最新代码，同时保证 master 分支是干净的。这样，我们每次切出的分支才是干净的，这会减少大量可能存在的 bug 引入与 冲突解决。

检查当前分支：

```shell
git branch
```

切回 master 分支

```shell
git checkout master
```

拉取最新代码

```shell
git pull --rebase
```

从 master 分支切出新分支

```shell
git checkout -b dev-jason
```

这时会已经切到了 dev-jason 分支，但是并没有声明当前分支所追踪的远程分支，则无法知道当前 diff。

所以跟踪远程 master 分支

```shell
git branch --set-upstream-to=origin/master dev-jason
#or
git branch -u origin master
```

这样就可以再新分支上开发并推送修改到远程 master 分支上了。

而多人协作时最常出现的情况就是在 push 远程分支上出现冲突，和前面提到的一样，这时因为远程分支已经进行了修改，但是我们切出的分支与远程分支已经分叉，需要做的就是将远程分支的修改拉到切出的分支上，使用 `git pull` 解决往冲突后，切出的分支实的修改实际就是基于当前更新后的远程 master 分支进行修改了。

### `git pull` 操作

`git pull` 命令先运行 `git fetch` 去下载相关仓库的内容。然后执行一个 `git merge` 操作，把远程的内容 ref 与 当前的 head merge, 并指向一个新的当地 merge commit。整个操作如同从图1

![bubble-diagram_01.svg](https://jason24-zeng.github.io/img/git-多人开发-与-分支管理/bubble-diagram_01.svg)

到图2

![bubble-diagram_02.svg](https://jason24-zeng.github.io/img/git-多人开发-与-分支管理/bubble-diagram_02.svg)

的整个流程。

相比 `git pull` 这个操作需要我们新更新一个 commit 以及 merge 切出去的分支的流程，我个人更喜欢用 `git pull --rebase` 这种换基的操作。这样做的好处是保证一个线性历史的追踪以及不必要的 merge commit。事实上，因为 --rebase 是一个非常常见的操作，所以我们可以直接设置一个 configuration, 保证使用 `git pull` 时，其实际操作是 `git pull --rebase`，这个操作是

```shell
git config --global branch.autosetuprebase always
```

与 merge 整个分支不同，rebase 把已提交的本地 commits 复制并写到了远程分支 commits 后面，这样就保证了历史 commit 的线性。其图表示为：

![bubble-diagram_03.png](https://jason24-zeng.github.io/img/git-多人开发-与-分支管理/bubble-diagram_03.png)
