---
title: 多机使用 Hexo
date: 2022-02-04 13:42:25
tags:
- Hexo
- Blog
cateogories:
- Hexo
categories: Hexo
copyright_author: "Jason Zeng"
cover: /img/hexo.png
top_img: /img/hexo.png
---

### Reference

[Git 工具 - 子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

[Git 分支 - 分支的新建与合并](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)

[多机使用Hexo博客](https://mindawei.github.io/2018/05/01/%E5%A4%9A%E6%9C%BA%E4%BD%BF%E7%94%A8Hexo%E5%8D%9A%E5%AE%A2/)

### 前言

因为工作原因，需要在新电脑上更新博客，为此，我特地研究了一下如何将源码传到博客上的方法。但是对于一些 prerequisite 的软件要求，还需要按照 [搭建 Hexo 博客](https://jason24-zeng.github.io/2022/01/14/%E6%90%AD%E5%BB%BAHexo-blog/) 里的提示下载安装。

### 思路

完成源码多机使用的关键是源码共享，为此，我们需要将 hexo 的根目录上传到一个远程仓库中，对其他机器可见，而其他有权限机器也能对其做修改。而 `git` 恰好可以满足我们的要求。

### 实现步骤

#### 创建贡献远程分支

##### 在博客根节点上初始化 git

`cd` 到相应目录，初始化根节点

```shell
git init
```

##### 创建并切换分支

```shell
git checkout -b <branch_name>
```

##### 连接远程分支到根目录

```shell
git remote add origin git@…<git_url>
```

##### 关联远程分支

```shell
git branch -u origin master
```

##### push 远程分支到相应的 branch 分支

```shell
git push origin <branch_name>
```

##### 切换跟踪分支

```shell
git branch -u orgin <branch_name>
```

##### 更新分支

```shell
# 方法 I
git add .
git commit -m "your commit"
# push
git push origin <branch_name>
```

#### 新主机使用博客操作

##### 拉取代码

##### 注意事项

注意上述操作可能会出现 submodule 的缺失问题，需要使用 `git submodule init` 等操作进行更新。主要原因是上传的根目录下，存在一些其他的 git 节点，这些节点没有被上传。
