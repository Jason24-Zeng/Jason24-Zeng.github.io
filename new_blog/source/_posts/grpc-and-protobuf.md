---
title: gRPC and protobuf
date: 2022-01-26 22:02:38
tags:
- gRPC
- ProtoBuf
categories: gRPC
cover: /img/grpc-and-protobuf/protocol-buffers.png
top_img: /img/grpc-and-protobuf/protocol-buffers.png
---

## gRPC

### 什么是 RPC?

RPC 是 Remote Procedure Call 的缩写，即远程过程调用，该调用包含了传输协议和编码协议等。无需额外的变成，就能实现一台计算机的程序为另一台计算机的程序调用的交互过程。我们常称之为 RPC 调用。从抽象的角度来说，RPC 不在本地执行，其都拥有三个特点：

- 需要事先约定调用的语义 -- Interface

- 需要网络传输

- 需要定义网络传输的数据结构

### 什么是 gRPC?

gRPC 是主要由 google 开发的开源，免费的 基于 Protobuf 开发的跨语言的 RPC 框架，其特点主要有：

1. 使用 ProtoBuf 进行数据编码，从而提升了数据的压缩。ProtoBuf 是一种 Interface Description Language (IDL) 借口描述行语言。

2. 使用 HTTP/2 带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特性传输协议，相比 HTTP 1.1 协议在移动设备上性能和空间占用上都有优化。

3. 同时在调用方 (Stub/Client) 与服务端 (Server) 使用协议约定文件，可以通过增加 protobuf 结构，为版本兼容留下缓冲空间。

### gRPC 基本组成

下图为一个简单的调用模型

![grpc_model](https://jason24-zeng.github.io/img/grpc-and-protobuf/grpc_concept_diagram.jpeg)

我们可以看到其主要的模块有：

- 客户端 (gRPC Stub)，通过程序调用方法，发起 RPC 调用

- 对请求信息事先 pb 对象序列化

- 服务端 (gRPC Server) 接受请求 Proto Request，解码 (反序列化) 请求内容结构，进行相关业务逻辑处理，并返回 Proto Responses.

- 从服务端传会客户端，也需要进行对象序列化，从而压缩传递空间。

- 客户端接收服务端的 Response，对其进行解码 (反序列化)。唤醒正在等待响应的客户端调用并返回响应结果。

### gRPC 的优势

1. 快速序列化，server 端和 stubs 端

2. 序列化结构较小，从而需要传输的带宽小

3. 基于 HTTP/2 协议进行设计，有显著的优势

4. 相比于 JSON、XML，定义更简单明了。

### gRPC 的缺点

1. Protobuf 序列化后的数据可读性查，无法想 HTTP/1.1 那样调试。

2. 需要额外的组建协助浏览器调用 gRPC 服务，同时对浏览器的支持是有限的。

3. 各大组建对 HTTP/2 的支持较差，即使支持，社区相关资料较少。

## ProtoBuf

### 什么是 ProtoBuf ?

Protocal Buffers (ProtoBuf) 是一种 IDL，具有可扩展的序列化结构化。它与平台，语言无关，常用于通信协议，数据存储等等。相比 JSON、XML 等结构，它更小、更快，收到广泛开发人员的青睐。

### ProtoBuf 基本语法

```protobuf
// 声明使用 proto3 语法，如果不声明，默认使用 proto2 语法
syntax = "proto3";

package helloworld;

// RPC 服务定义
service Greeter {
    // 定义了一个 RPC 方法，叫做 SayHello，入参为 HelloRequest 结构体，出参为 HelloReply 结构体
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 消息体，
message HelloRequest {
    // 消息体中的字段，定义了字段类型，字段名称，以及 idx
    string name = 1;
}


message HelloReply {
    string message = 1;
}
```

proto 文件通常以 `.proto` 后缀结尾，通常进行编译并生成对应语言的 proto 文件。根据 Protobuf 编辑器选择的语言或者调用的插件情况的不同，生成相对应的 Service Interface Code 和 Stubs

#### Proto 编译

前面已经提到，protobuf 支持多语言，而语言之间的切换实际是通过使用一个编译器的不同插件完成的。这个编译器就是 `protoc`，对 `.proto` 文件进行编译

##### `protoc` 安装

为了安装 `protoc` ，我们依次执行以下安装 command

```shell
wget https://github.com/google/protobuf/releases/downloads/v3.11.2/protobuf-all-3.11.2.zip
unzip protobuf-all-3.11.2.zip && cd protobuf-3.11.2/
./configure
make
make install
```

安装完成后，可以通过执行 `protoc --version` 检查是否安装成功

##### `protoc-gen-go` 插件安装

假设我们已经完成了上面关于 `protoc` 编译器的安装，但是仅仅有这个编译器还是不够的。前面提到，针对不同的语言，我们需要调用不同的 `protoc` 插件，从而完成相关语言 `.proto` 文件的转换。

因此，我们主要执行以下命令：

```shell
# Instal a specific version.
go install example.com/cmd@v1.2.3

# Install the highest available version.
go install example.com/cmd@latest
```

需要注意：`Go 1.17` 之后，使用 `go get` 去安装插件的方式不再被推荐，取而代之的是使用 `go install` 方法，因为这个原因，我刚开始使用 

```go
go get -u google.golang.org/protobuf/cmd/protoc-gen-go
```

会有下述错误

```shell
can't load package: package google.golang.org/protobuf/cmd/protoc-gen-go: cannot find package "google.golang.org/protobuf/cmd/protoc-gen-go" in any of ...
```

根据这个社区讨论  [cannot find package "google.golang.org/protobuf/cmd/protoc-gen-go"](https://stackoverflow.com/questions/62190610/cannot-find-package-google-golang-org-protobuf-cmd-protoc-gen-go)，我们可以有一些其他的解决方法，但是，比较推荐的方式还是使用 `go install`

在安装以后，还需要给 golang 添加路径（否则会出现报错：[protoc-gen-go: program not found or is not executable](https://stackoverflow.com/questions/57700860/protoc-gen-go-program-not-found-or-is-not-executable)）：

1. 执行 `vim ~/.bash_profile`

2. 添加
   
   ```shell
   export GO_PATH=$HOME/go
   export PATH=$PATH:/$GO_PATH/bin
   ```

3. 执行 `source ~/.bash_profile`

另一种方法是直接将二进制文件目录 `bin` 移到默认路径上。不过这种方法需要当前执行者具有移到 local 路径的权限。

```shell
mv ~/go/bin/protoc-gen-go /usr/local/go/bin
```

## Reference

1. [Go 语言编程之旅](https://golang2.eddycjy.com/posts/ch3/01-simple-grpc-protobuf/)

2. [gRPC系列(一) 什么是RPC？](https://zhuanlan.zhihu.com/p/148139089#:~:text=gRPC%E6%98%AF%E4%B8%80%E6%AC%BERPC,%E5%85%BC%E5%AE%B9%E7%95%99%E4%B8%8B%E7%BC%93%E5%86%B2%E7%A9%BA%E9%97%B4)

3. [Protocol Buffer Basics: C++](https://developers.google.com/protocol-buffers/docs/cpptutorial)
