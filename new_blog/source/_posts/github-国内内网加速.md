---
title: github 国内内网加速
date: 2022-03-24 00:21:55
tags: 
- Github
- Host
category:
- Dev Tools
- Version Control System
cover: /img/git-多人开发-与-分支管理/git-flow.webp
top_img: /img/git-多人开发-与-分支管理/git-flow.webp
---

## 前言

最近 VPN 被墙，这段时间博客的更新都受到了限制。还好最近使用了新的 VPN，但是却只能在 chrome 上连接网页，登录网站使用。这依然无法让我使用 git 拉取和 push 博客。直到今天（2022-03-23），才通过筛选网络搜索的各种方法，发现了现在依然能够正常使用的，快速 pull 与 push git commit 的途径。

而这个方法，就是使用 HOST 调整网站连接端口。

## 使用步骤

假设我们目前已经有了一个 git 本地分支（如果构建本地分支等操作不在本博客范围内），我们要做的就是保证其正常使用我们的账号 push 与 pull 本地贮存的 commit.

### 检查不同地区访问时延

登录网站 [“github.com”国际网站测速结果--国际网站测速-站长工具](http://tool.chinaz.com/speedworld/github.com) 输入 `github.com` 查看分析当前不同监测点解析 IP 的访问时延，可以看到总耗时最短的为新加坡的站点

![时延](https://jason24-zeng.github.io/img/github-国内内网加速/2022-03-24-00-33-58-image.png)

我们记住它的解析 ip: `20.205.243.166`，这个 ip 我们后面会用来作为访问 github 的 host 文件对应 ip

### 修改 host 文件

如果是 mac 电脑，修改 `/etc/hosts` 文件

```shell
su vim /etc/hosts
```

添加如下命令行到 hosts 文件中

```shell
20.205.243.166▸ github.com #/t
```

### 重启

重启 Terminal 之后，再使用 `git pull` 与 `git push` 等指令，就可以快速完成。
