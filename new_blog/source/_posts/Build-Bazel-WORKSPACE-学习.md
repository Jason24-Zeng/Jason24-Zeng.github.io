---
title: Bazel WORKSPACE 学习
date: 2022-09-12 23:25:39
tags: 
- Bazel
- BUILD
categories:
- Coding Language
- C++
top_img: /img/bazel_tag.png
cover: /img/C-usable-example/C.png
---

## 场景 I

### `new_local_repository` 使用

在引入第三方库 openssl 时，我们在 `WORKSPACE` (`~/bazel-workspace`) 中通过加入 `new_local_repository` 的方式引入库。

```shell
new_local_repository(
    name = "openssl",
    path = "/usr",
    build_file = "depBUILD/openssl.BUILD",
)
```

首先看看这个 `new_local_repository` [参数](https://docs.bazel.build/versions/main/be/workspace.html#new_local_repository:~:text=use%20piano.jar.-,Arguments,-Attributes)的介绍：

```shell
new_local_repository(name, build_file, build_file_content, path, repo_mapping, workspace_file, workspace_file_content)
```

该函数允许一个本地目录被变成 bazel 的仓库，这意味着现在的仓库能定义和使用文件系统任何地方的 target。

这个规则通过创建一个 `WORKSPACE`文件和「包含到`BUILD`文件和路径的 symlinks」的子目录，最终创建一个 Bazel 仓库。对已经包含一个 `WORKSPACE`  文件和 `BUILD` 文件的目录，我们可以直接使用 `local_repository`规则。

针对这里的情况，我们会创造一个 `@openssl` 的库，这个库与符号链接`symlink` 到 /usr 路径。我们的 Targets 可以通过加诸如`@openssl:crypto` 到目标依赖 (target's dependencies) 的方式依赖这个库。

参数 `build_file` 是相对于主仓库的路径或绝对路径。该路径的 `BUILD` 文件 (`~/bazel-workspace/depBUILD/openssl.BUILD`) 为：

```shell
package(
    default_visibility=["//visibility:public"]
)

config_setting(
    name = "macos",
    values = {
        "cpu": "darwin",
    },
    visibility = ["//visibility:private"],
)

cc_library(
    name = "crypto",
    srcs = select({
        ":macos": ["lib/libcrypto.dylib"],
        "//conditions:default": []
    }),
    linkopts = select({
        ":macos" : [],
        "//conditions:default": ["-lcrypto"],
    }),
)

cc_library(
    name = "ssl",
    hdrs = select({
        ":macos": glob(["include/openssl/*.h"]),
        "//conditions:default": []
    }),
    srcs = select ({
        ":macos": ["lib/libssl.dylib"],
        "//conditions:default": []
    }),
    includes = ["include"],
    linkopts = select({
        ":macos" : [],
        "//conditions:default": ["-lssl"],
    }),
    deps = [":crypto"]
)
```

由此，继续介绍以上规则的含义

### `package`

[Bazel package function](https://bazel.build/reference/be/functions)

```shell
package(default_deprecation, default_testonly, default_visibility, features)
```

该函数声明对包中后续的每一个规则的 metadata (元数据) 均适用。它在一个包内（BUILD 文件）中只能使用最多一次。

package 函数应该在所有 load 声明完成之后，在任何规则之前马上加上。

### `config_setting`

[Bazel config setting function](https://bazel.build/reference/be/general#config_setting)

```shell
config_setting(name, constraint_values, define_values, deprecation, distribs, features, flag_values, licenses, tags, testonly, values, visibility)
```

为了触发可配置属性，`config_setting` 被使用来匹配一个期望的配置状态。（被表达为 编译 flags 或平台约束）。可以理解为，对不同平台，可能我们的编译参数不一致，而 `config_setting` 和 `select` 的 combo 使用帮助我们完成这一点。

#### 例子

第一个例子，它匹配任何设置 `--compilation_mode = opt` 或者 `-c opt`的编译，（可能显式得出现在命令行或者 `.bazelrc` 文件）

```shell
config_setting(
    name = "simple",
    values = {"compilation_mode": "opt"}
)
```

第二个例子，它匹配任意编译，该编译以 ARM 为目标，并且应用自定义的设置`FOO=bar`。（比如 `bazel build --cpu=arm --define FOO-bar ...`）：

```shell
config_setting(
    name = "two_conditions",
    values = {
        "cpu": "arm",
        "define": "FOO=bar"
    }
)
```
