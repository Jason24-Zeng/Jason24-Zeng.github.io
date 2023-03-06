---
title: Docker Intro (I)
date: 2022-01-29 18:05:54
tags:
- Docker
categories:
- Dev Tools
- Dev Machine
cover: /img/Docker-Intro-I/docker-top.png
top_img: /img/Docker-Intro-I/docker-top.png
---

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到可移植的镜像中。这样的镜像可以被在任意的 Linux 或 Windows 等包含特定输入输出规则的机器上。它完全使用沙箱机制，相互之间无任何接口。

### Docker 安装

#### 命令行 Homebrew 安装

通过 homebrew 下载并安装到应用目录

```shell
brew install --cask --appdir=/Applications docker
```

#### Docker Desktop for Mac 安装包安装

Docker Desktop for Mac 是 Docker 的一种桌面管理 IDE，用于在 Mac 上构建，调试和测试 Dockerized 应用程序。它是一个完整的开发环境，与 Mac OS Hypervisor 框架，网络与文件系统深度继承，是在 Mac 上运行 Docker 的最快，最可靠的方法。

在Docker官方网站下载安装文件：[https://hub.docker.com/editions/community/docker-ce-desktop-mac](https://link.zhihu.com/?target=https%3A//yq.aliyun.com/go/articleRenderRedirect%3Furl%3Dhttps%253A%252F%252Fhub.docker.com%252Feditions%252Fcommunity%252Fdocker-ce-desktop-mac)  
下载 Docker.dmg安装文件，直接双击安装完成就可以了。

### Docker 更新镜像源

运行镜像拉取时，可能因为网络原因出现下面 ERROR 

```textile
ERROR: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

为了解决这个原因，我们可以手动指定镜像源，在 Mac OS 系统中，更新镜像源的文件为 

`~/.docker/daemon.json` 文件，只用加入 以下 json 配置即可更换镜像源

```json
{
    "registry-mirrors": [
        "https://registry.docker-cn.com"， // 中国区官方镜像
        "http://hub-mirror.c.163.com",  // 网易镜像
        "https://docker.mirrors.ustc.edu.cn"  // 中科大镜像
    ]
}
```

如果通过 Homebrew 安装的 Docker，可以尝试点击右上角的🐳图标 > Preference >  Docker Engine 去直接修改那个 configuration 文件

### Docker 容器内安装 vim 等命令行参数

我们登入 docker 的容器，发现没办法使用 vim 等命令去修改 config 文件或者编辑其他文件，会提示：`vim: command not found`

为了 solve 这个问题，我们可以进入容器并安装 vim 命令

#### 可以使用 `apt-get` 的 `sudo` 权限安装

```shell
apt-get update
apt-get install vim
```

#### 可以使用 `yum` 的 `root` 权限安装

```shell
# 进入docker 是开启 root
docker exec -it --user root 473f6e871544 /bin/bash
# 更新下列配置，否则会报错
# Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-Linux-* &&\
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-Linux-*
# 更新 yum
yum upgrade -y
# 下载安装 vim
yum install vim
```

#### 安装 `top`、`ps` 等命令行

```shell
apt-get install procps
```

不过需要注意的是，最好不要在 docker 容器中修改文件，而是将经常修改的文件挂载到宿主机上，这样避免重启时无法回复修改。

### Docker 挂载本地目录

Docker 支持把一个宿主机上的目录挂载到镜像里，这样我们就不需要通过修改镜像内的文件去修改配置了。

```shell
docker run -d -v /usr/local/es:/usr/share/elasticsearch/data -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 --name es elasticsearch
```

### Docker 相关问题与解决

#### Docker 无法通过指令停止

今天出现了一个无法通过 stop 与 kill 等相关的指令停止或终止容器的问题。

首先是在已经有一个容器正常运行的基础上，重新 `docker-compose up -d` 了一个新容器，调用了同一个端口作为映射，导致错误

```shell
docker-compose up -d
-----------------------------
Creating network "elasticsearch_default" with the default driver
Creating volume "elasticsearch_es-data" with local driver
Creating elasticsearch_01 ...

ERROR: for elasticsearch_01  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)

ERROR: for elasticsearch_01  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)
ERROR: An HTTP request took too long to complete. Retry with --verbose to obtain debug information.
If you encounter this issue regularly because of slow network conditions, consider setting COMPOSE_HTTP_TIMEOUT to a higher value (current value: 60).
```

这时候通过 `docker ps -a` 可以看到，容器 STATUS 是 CREATED，但是没办法启动，因为接口不对。所以想要停止之前的容器，然后再启动新容器的服务。这时候发现新容器停不下来了，看容器内服务信息`docker logs container-id`为

```textile
{"type": "server", "timestamp": "2022-01-31T07:15:51,250Z", "level": "ERROR", "component": "i.n.u.c.D.rejectedExecution", "cluster.name": "docker-cluster", "node.name": "8d619a059150", "message": "Failed to submit a listener notification task. Event loop shut down?", "cluster.uuid": "S33uPEbYT4iKRFfqRiul-Q", "node.id": "To5wb_cQRg-BheySS5Nliw" ,
"stacktrace": ["java.util.concurrent.RejectedExecutionException: event executor terminated",
"at io.netty.util.concurrent.SingleThreadEventExecutor.reject(SingleThreadEventExecutor.java:923) ~[netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.SingleThreadEventExecutor.offerTask(SingleThreadEventExecutor.java:350) ~[netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.SingleThreadEventExecutor.addTask(SingleThreadEventExecutor.java:343) ~[netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.SingleThreadEventExecutor.execute(SingleThreadEventExecutor.java:825) ~[netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.SingleThreadEventExecutor.execute(SingleThreadEventExecutor.java:815) ~[netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.DefaultPromise.safeExecute(DefaultPromise.java:842) [netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.DefaultPromise.notifyListeners(DefaultPromise.java:499) [netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.DefaultPromise.setValue0(DefaultPromise.java:616) [netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.DefaultPromise.setFailure0(DefaultPromise.java:609) [netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.util.concurrent.DefaultPromise.setFailure(DefaultPromise.java:109) [netty-common-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.channel.DefaultChannelPromise.setFailure(DefaultChannelPromise.java:89) [netty-transport-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.channel.AbstractChannelHandlerContext.safeExecute(AbstractChannelHandlerContext.java:998) [netty-transport-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.channel.AbstractChannelHandlerContext.write(AbstractChannelHandlerContext.java:796) [netty-transport-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.channel.AbstractChannelHandlerContext.writeAndFlush(AbstractChannelHandlerContext.java:758) [netty-transport-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.channel.DefaultChannelPipeline.writeAndFlush(DefaultChannelPipeline.java:1020) [netty-transport-4.1.66.Final.jar:4.1.66.Final]",
"at io.netty.channel.AbstractChannel.writeAndFlush(AbstractChannel.java:311) [netty-transport-4.1.66.Final.jar:4.1.66.Final]",
"at org.elasticsearch.http.netty4.Netty4HttpChannel.sendResponse(Netty4HttpChannel.java:34) [transport-netty4-client-7.16.2.jar:7.16.2]",
"at org.elasticsearch.http.DefaultRestChannel.sendResponse(DefaultRestChannel.java:134) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.rest.RestController$ResourceHandlingHttpChannel.sendResponse(RestController.java:588) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.rest.action.RestActionListener.onFailure(RestActionListener.java:55) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.rest.action.RestActionListener.onResponse(RestActionListener.java:40) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.action.support.TransportAction$1.onResponse(TransportAction.java:88) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.action.support.TransportAction$1.onResponse(TransportAction.java:82) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.action.ActionListener.completeWith(ActionListener.java:447) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.action.support.nodes.TransportNodesAction.newResponseAsync(TransportNodesAction.java:181) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.action.support.nodes.TransportNodesAction.newResponse(TransportNodesAction.java:156) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.action.support.nodes.TransportNodesAction$AsyncAction.lambda$finishHim$0(TransportNodesAction.java:295) [elasticsearch-7.16.2.jar:7.16.2]",
"at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingRunnable.run(ThreadContext.java:718) [elasticsearch-7.16.2.jar:7.16.2]",
"at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136) [?:?]",
"at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635) [?:?]",
"at java.lang.Thread.run(Thread.java:833) [?:?]"] }
```

出现问题的最主要原因可能是，新创建的容器的某一个配置抢占了旧容器的端口，也阻止了旧容器重新生成旧名称的新容器，因为系统认为旧容器依然存在。

首先我找到新创建的异常容器，使用 `docker rm container-id`删除掉了，但是依然 stop 旧容器，报错是 `Cannot kill container: : tried to kill container, but did not receive an exit event`。

最后没有办法，直接重启了 docker desktop，就看到旧容器已经 dead 了，然后删除容器即可。

### Reference

[一篇不一样的docker原理解析](https://zhuanlan.zhihu.com/p/22382728)

[一篇不一样的docker原理解析 提高篇](https://zhuanlan.zhihu.com/p/22403015)
