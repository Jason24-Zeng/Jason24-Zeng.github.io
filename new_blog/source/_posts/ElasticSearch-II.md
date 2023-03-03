---
title: ElasticSearch (II) -- docker 启动 + docker-compose
date: 2022-01-30 18:22:42
tags: 
- ElasticSearch
- Apache
categories:
- Dev Tools
- ElasticSearch
cover: /img/ElasticSearch/elastic_search_top.png
top_img: /img/ElasticSearch/elastic_search_top.png
---

## 安装与运行 Elasticsearch

假设已经下载 Docker、Elasticsearch 和 Kibana，接下来讲解在 docker 里面启动 Elasticsearch 和 Kibana，并对 Elasticsearch 集群启用多个节点。

### Docker 中启动单节点 Elasticsearch

使用如下指令以实例化 ElasticSearch 服务

```shell
 docker run --name es -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.16.2
```

以上将 9200 端口与 9300 端口映射到 docker 相应端口，如果不做端口映射，浏览器无法访问 elasticsearch 服务。其中 9200 为供 http 访问端口，9300 为供 tcp 访问端口。注意 `discovery.type=single-node`  指令，对于单机只生成一个节点的情况而言，这个必须指明，否则 docker 启动 elasticsearch 会闪退，相关报错为：`the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured`。

如果想要实现数据持久化，则需要通过 -v 参数将 docker 宿主机 (host) 上的目录 mount 到 ElasticSearch 容器里。相关指令如下（`/usr/local/es` 为宿主机的目录地址）：

```shell
docker run --name es -d -v /usr/local/es:/usr/share/elasticsearch/data -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.16.2
```

需要注意的是，宿主机的目录需要赋权，否则会报错：failed to bind service AccessDeniedException

```shell
chmod 777 /usr/local/es
```

执行 `docker ps` 查看容器是否成功运行。在打开浏览器访问 `localhost:9200`，如果展现如下，则说明服务启动成功。

![local_host.png](https://jason24-zeng.github.io/img/ElasticSearch-II/local_host.png)

如果需要可视化管理界面，可以通过 Chrome 浏览器安装插件 ElasticSearch Head. 

![elasticsearch_head.png](https://jason24-zeng.github.io/img/ElasticSearch-II/elasticsearch_head.png)

则最终的展示为

![elasticsearch_head2.png](https://jason24-zeng.github.io/img/ElasticSearch-II/elasticsearch_head2.png)

### ElasticSearch 测试环境配置修改

配置文件的修改是很有必要的，因为原始的配置文件可能在网络或者资源环境要求上无法满足/overqualify 了测试环境的要求。

#### 修改 ES 配置文件

进入容器后，需要修改 es 配置文件，设置宿主机的 ip 地址为任意网络均可访问。

```shell
vim config/elasticsearch.yml
# 修改如下 host network ip address
network.host: 0.0.0.0
```

#### 修改启动参数

如果 Elasticsearch 中的 network.host 不是 localhost 或者 127.0.0.1 的话，该容器环境会被认为是生产环境，这个环境要求的一些内存等比较高，测试环境不一定满足，因此需要修改一些配置。

修改 jvm 启动参数

```shell
vim config/jvm.options
# 修改如下配置，将 heap 内存启动参数和最大参数从 4g 修改成 256m
-Xms256m
-Xmx256m
```

设置一个进程在 VMAs (Virtual Memory Areas) 创建内存映射的最大数量。这个操作需要使用 root 用户去操作。

```shell
 vim /etc/sysctl.conf
 # 加上如下配置
 vm.max_map_count=655360
 # 记得需要用 root 权限生效该配置。不过也可能因为虚拟机是 OpenZV 而无法修改
 sysctl -p
```

以 root 权限登入容器 

```shell
docker exec -it --user root f9c23e8de222 /bin/bash
```

任何用户组的权限都可以这样切换。如果忘记了用户组名，可以在容器中输入 `id`, 可以从 `groups` 中看到组内所有用户。

如果想要使用 jps、jstack 等 jvm 监控命令，则可能需要换一个 JDK 镜像，因为 docker 中使用的 JDK 镜像是精简版，没有这些额外的 JDK 辅助工具包

### Docker-Compose 使用

通过上面这些指令去启用 docker 会比较麻烦，主要是后续需要修改一些配置，比如 mount 文件与内存映射最大数量等，这些需要在启动的时候就执行成功，且给予相应的权限，否则进入容器后会因为容器内文件的只读性质而无法更新。同样的，如果写一条冗长的指令满足上述要求的同时，却失去了命令行的简洁直观。这种情况下，提前在宿主机上设置好配置文件，在启动容器时就配置好容器内的一些参数，会是非常必要的方法。而 Docker-Compose 恰好可以承担这样的任务。

对于下载和安装了 Docker Desktop 或者 Decker for Mac 等的 Mac 用户，我们就不必再单独安装 docker-compose 了。而对于 Linux 用户，则需要如下操作：

#### Linux 下 Compose 的安装

1. 下载 Docker Compose 稳定版本
   
   ```shell
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

2. 给予 binary 可执行权限
   
   ```shell
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. 构建软链，供执行命令调用
   
   ```shell
   sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
   ```

4. 测试是否安装成功
   
   ```shell
    docker-compose --version
   ```

#### 配置启动文件 `docker-compose.yaml`

通过配置启动文件，我们可以将一些参数从宿主机直接传到容器。

##### 单 es 节点 + kibana 配置

```yaml
version: "3.8"

services:
  elasticsearch:
    image: elasticsearch:7.16.2
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      # - cluster.initial_master_nodes=elasticsearch
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 9200:9200
    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data
  kibana:
    image: kibana:7.16.2
    container_name: kibana_1
    depends_on:
      - elasticsearch
    ports:
      - 5601:5601

volumes:
  es-data:
    driver: local
```

注意：单节点 `single-mode` 模式下，`cluster.initial_master_nodes` 是不被允许设置的。

##### 单 es 节点 + kibana 配置 + 使用网络

```yaml
version: "3.8"

services:
  elasticsearch:
    image: elasticsearch:7.16.2
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      # - cluster.initial_master_nodes=elasticsearch
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 9200:9200
      - 9300:9300
    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/logs:/usr/share/elasticsearch/logs
    networks:
      - es-net
  kibana:
    image: kibana:7.16.2
    container_name: kibana_1
    depends_on:
      - elasticsearch
    ports:
      - 5601:5601
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - I18N_LOCALE=zh-CN
      - xpack.monitoring.ui.container.elasticsearch.enabled=false
    volumes:
      - ./kibana/kibana.yml:/usr/share/kibana/config/kibana.yml:rw
    networks:
      - es-net
networks:
  es-net:
    external: true
```

需要注意的是，我们这里 `volumes` 不知用到了 data，还需要在当前目录 `touch  -e ./kibana/kibana.yml` 和 `mkdir -p ./elasticsearch/logs`

另外需要注意

`external` is to use an existing network. If you want compose to make networks you simply do:

```shell
networks:
  network1:
```

##### 多 es 节点 + kibana 配置

这里我们用三个节点来做简单配置。需要根据节点数量配置 memory 的大小，刚开始我只设置了 1 G 的内存，因为启动时就需要 1.5 G 内存，结果容器直接启动失败，且无相关日志产出。

```yaml
version: '3.8'
services:
  es01:
    image: elasticsearch:7.16.2
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./es01/data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic

  es02:
    image: elasticsearch:7.16.2
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./es02/data:/usr/share/elasticsearch/data
    networks:
      - elastic

  es03:
    image: elasticsearch:7.16.2
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./es03/data:/usr/share/elasticsearch/data
    networks:
      - elastic

  kib01:
    image: kibana:7.16.2
    container_name: kib01
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: http://es01:9200
      ELASTICSEARCH_HOSTS: '["http://es01:9200","http://es02:9200","http://es03:9200"]'
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```

#### 启动容器

在启动容器之前，还需要根据配置提前设置到 `volumes`。比如在单节点模式下，我们进入一个目录，假设是 `$Home`，那么我们需要

1. 提前创建宿主机上的映射目录 `mkdir -p elasticsearch/data`

2. 给予相关目录 777 权限 `chmod -R 777 elasticsearch`

设置好之后，在`$Home` 目录下执行如下命令 

```shell
docker-compose up -d
```

docker 就会去访问`$Home`目录下 `docker-compose.yaml` 等相关配置文件。根据配置文件去从网络或者本地拉取镜像，新建并使用镜像启动服务等。而上面的 `-d` 是 detach 的意思，也就是不仅如此容器。

可以通过 `docker ps -a` 等指令查看容器的相关状态，比如是否是 `up` 拉起的状态，或者因为某一些原因`Exited`退出等。拉起一段时间后，可以通过访问 `localhost:5601` 查看 kibana 服务是否正常。

#### 关闭容器等相关操作

停止当前目录下 `docker-compose.yaml` 文件启动的相关容器，使用 `docker-compose stop` 或 `docker-compose kill` 停掉 docker。这两者的主要区别是 `stop` 支持优雅退出，也就是会先接受 SIGTERM 请求，做一些预处理工作保存状态，10 s 后再执行 SIGKILL 请求。而 `kill` 则是直接执行 SIGKILL 请求对应的操作。

要查看当前`docker-compose.yaml` 启用的容器的状态，可以使用 `docker-compose ps`

关闭服务后的容器的启动可使用 `docker-compose start` 指令，而如果因为更新配置要重启服务，可以使用 `docker-compose restart` 指令。

如果想要看当前 docker 运行情况，可以通过指令`docker-compose top` 完成

## Reference

[Docker部署ElasticSearch及使用](https://juejin.cn/post/6844904202204872711)

[ElasticSearch 6.x 增删改查操作汇总 及 python调用ES中文检索实例_私人天地-程序员信息网](https://www.i4k.xyz/article/Dooonald/87931435)

[ElasticSearch Server with Docker Compose: java.nio.file.AccessDeniedException: /usr/share/elasticsearch/data/nodes](https://stackoverflow.com/questions/65295961/elasticsearch-server-with-docker-compose-java-nio-file-accessdeniedexception)

[docker-compose部署ElasticSearch集群 | January](https://zysite.top/archives/elasticsearch-docker-compose-install)

[elasticsearch:7.4.2 的docker compose文件](https://blog.csdn.net/u011790603/article/details/105227925)
