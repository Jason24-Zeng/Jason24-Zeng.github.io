---
title: git 多用户配置
date: 2022-03-02 21:46:20
tags:
- Git
- Devops
category:
- Dev Tools
- Version Control System
cover: /img/git-多人开发-与-分支管理/git-flow.webp
top_img: /img/git-多人开发-与-分支管理/git-flow.webp
---

## 前言

之前会经常碰到这样一个问题：在一台电脑上，我们可能会需要开发公司的代码，这需要公司 gitlab 的相关用户，而这台电脑同时又可以作为自己 github 上博客的本地仓库开发地。这时候我们就需要区别不同的 git 用户，并做到轻松切换。否则，我们会经常出现无读写权限的问题。这篇博客主要讨论的，便是如何轻松在一台机器上使用多个 git 用户的方法。

### 移除 Git 全局配置

如果我们 git 配置了全局的 username 或者 email 等，当这个配置与目标 git 所需配置不符时，就可能出现无权限读写的问题。因此，为了后面能使用多个 git。我们需要首先移除全局的 git 配置。步骤如下

1. 打开任意 command line terminal，比如 iterm2

2. 输入 `git config --list` ，我们可以看到我们的全局配置

3. 对已有的全局配置，比如 name, email, 和 password 等全局配置，我们使用一下指令去除 
   
   ```shell
   git config --global --unset user.name
   git config --global --unset user.email
   git config --global --unset user.password
   ```

### 生成不同 Git 公钥

假设我们有两个用户如下

| Git平台   | username 用户名 | email邮箱                                                     | hostname 主机名    |
| ------- | ------------ | ----------------------------------------------------------- | --------------- |
| 公司私有Git | yourname     | [yourname@git.company.com](mailto:yourname@git.company.com) | git.company.com |
| Github  | nickname     | [nickname@github.com](mailto:nickname@github.com)           | github.com      |

我们分别生成两个公钥，到不同的文件

```shell
ssh-keygen -t rsa -C "yourname@git.company.com" -f  ~/.ssh/id_rsa_gitlab
ssh-keygen -t rsa -C "nickname@github.com" -f  ~/.ssh/id_rsa_github
```

### 公钥添加 && 私钥登记

将不同的公钥（比如 `id_rsa_gitlab`，`id_rsa_github`）添加到各个平台，以便 ssh 时能够 Authentization。

同时，将私钥使用代理模式登记到本地

```shell
# 启用代理模式
ssh-agent bash

# 将 Gitlab 私钥添加到本地
ssh-add ~/.ssh/id_rsa_gitlab

# 将 Github 私钥添加到本地
ssh-add ~/.ssh/id_rsa_github

# 验证用户
ssh -T git@git.company.com
ssh -T git@github.com
```

最终，可以通过执行 `ssh-add -l` 验证登记

### 配置 ssh 的 config 文件

config 文件目录在 `~/.ssh` 创建 `config` 文件

```shell
# 公司gitlab
Host gitlab
    HostName git.company.com
    User yourname
    IdentityFile ~/.ssh/id_rsa_gitlab

# GitHub
Host github
    HostName github.com
    User nickname
    IdentityFile ~/.ssh/id_rsa_github
```

该文件分为多个用户配置，每个用户配置包含以下几个配置项：

- Host：Git 平台的别名，clone仓库时，可以替代 HostName 来使用
- HostName：Git 平台的域名（PS：IP 地址应该也可以）
- User：邮箱或用户名
- IdentityFile：私钥路径

### 仓库配置

每次 clone 后，需要给每个仓库配置用户名与邮箱，这样仓库在提交代码是，程序可以知道提交的服务器是哪个，也就清楚使用哪一个密钥。

```shell
git clone git@github.com:personal-username/xxx.git
# 配置用户名
git config user.name "nickname"
# 配置邮箱
git config user.email "nickname@github.com"
```

如果 repo 还未 clone 到本地，可以修改 clone 命令

```shell
git clone git@personal:xxx.git
```

注意这个 personal 就是 `config` 文件的 Host 的名字

### Reference

[通过 ssh config 配置 Git 多账户 SSH 登录 | 成长自习室](https://hanpanpan200.github.io/2019/10/14/setup-multiple-git-accounts-by-ssh-config/)

[Git 配置多用户](https://emmxxx.com/archives/18)

[一台电脑，两个及多个git账号配置](https://www.1024sou.com/article/9523.html)
