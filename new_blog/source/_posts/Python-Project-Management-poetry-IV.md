---
title: Python Project Management - poetry (IV)
date: 2023-03-09 22:55:58
tags: 
- Poetry
- Project Management
- Configuration
categories:
- Coding Language
- Python
passwd: 0418
cover: /img/python_poetry.webp
top_img: /img/python_poetry.webp
---

# 配置

Poetry 可以通过 `config` 指令完成配置，或者直接在 `config.toml` 文件中完成，当我们第一次跑这个指令时，该文件会被自动创建。该文件能在一下路径下被发现。

- macOS: `~/Library/Preferences/pypoetry`

- Windows: `%APPDATA%\pypoetry`

对于 Unix 系统，我们遵守 XDG 规范，并且支持 `$XDG_CONFIG_HOME`，也就是默认的 `~/.config/pypoetry`

## 局部配置

Poetry 也提供对项目定制化配置的能力，只需要我们对 `config` 指令传递 `--local` 选项。

```shell
poetry config virtualenvs.create false --local
```

我们 Poetry  应用的局部配置能被存储在 `poetry.toml` 文件内，它会和 `pyproject.toml` 局别开。需要注意将该文件签入存储库的动作，因为它可能包含用于定制性的信息甚至敏感信息。

## 列举当前配置

想要列举当前的配置，我们可以给 `config` 指令传递 `--list` 选项

```shell
poetry config --list
```

它可能会给我们一些如下的配置信息

```toml
cache-dir = "/path/to/cache/directory"
virtualenvs.create = true
virtualenvs.in-project = null
virtualenvs.options.always-copy = true
virtualenvs.options.no-pip = false
virtualenvs.options.no-setuptools = false
virtualenvs.options.system-site-packages = false
virtualenvs.path = "{cache-dir}/virtualenvs"  # /path/to/cache/directory/virtualenvs
virtualenvs.prefer-active-python = false
virtualenvs.prompt = "{project_name}-py{python_version}"
```

## 展示单一配置字段

如果我们想要看一个指定配置的值，我们可能在 `config` 指令后面跟上该配置的名字

```shell
poetry config virtualenvs.path
```

## 增加或者更新一个配置

想要增加或者改变一个配置设置，我们能在配置的名字后传递一个值

```shell
poetry config virtualenvs.path /path/to/cache/directory/virtualenvs
```

## 移除一个指定配置

如果我们想要移除一个之前设定的配置，我们可以使用 `-unset` 选项

```shell
poetry config virtualenvs.path --unset
```

该设置会取回它的默认值

## 使用环境变量

有时，尤其当用 CI 工具来使用 Poetry 时，使用环境变量会跟容易，并且也不必执行配置指令。

Poetry 支持这种做法，并且任何配置都能通过使用环境变量设定

当然，环境变量必须以 `POETRY_` 开头，并且它需要包含设置的大写名字，同时用下划线代替点和破折号。举例如下：

```shell
export POETRY_VIRTUALENVS_PATH=/path/to/virtualenvs/directory
```

这种方法在隐私设置上也使用，比如凭据：

```shell
export POETRY_HTTP_BASIC_MY_REPOSITORY_PASSWORD=secret
```

## 默认目录

Poetry 会使用如下默认目录

### 配置目录

- Linux：`$XDG_CONFIG_HOME/pypoetry` 或 `~/.config/pypoetry`

- Window：`%APPDATA%\pypoetry`

- MacOs: `~/Library/Preferences/pypoetry`

我们可以通过设定 `POETRY_CONFIG_DIR` 环境变量的方法覆写配置目录

### 数据目录

- Linux：`$XDG_DATA_HOME/pypoetry` 或 `~/.local/share/pypoetry`

- Window：`%APPDATA%\pypoetry`

- MacOs: `~/Library/Application Support/pypoetry`

我们能通过设定 `POETRY_DATA_DIR` 或者 `POETRY_HOME` 环境变量的方式覆写数据目录。如果 `POETRY_HOME` 被设定，他会被给定一个更高的优先级。

### 缓存目录

- Linux：`$XDG_CACHE_HOME/pypoetry` 或 `~/.cache/pypoetry`

- Window：`%LOCALAPPDATA%\pypoetry`

- MacOs: `~/Library/Caches/pypoetry`

我们能通过设定 `POETRY_CACHE_DIR` 环境变量的方式覆写缓存目录

## 可用设置

### `cache-dir`

类型：`string`

含义：被 Poetry 使用的缓存目录的路径

默认值为如下目录之一

- MacOS: `~/Library/Caches/pypoetry`

- Windows:`C:\Users\<username>\AppData\Local\pypoetry\Cache`

- Unix: `~/.cache/pypoetry`

### `experimental.system-git-client`

类型: `boolean`

默认值：`false`

使用系统 git 客户端后端执行 git 相关任务

Poetry 默认使用 `dulwich` 来执行 git 相关任务而不依赖一个 git 客户端的可用性

如果我们碰到任何与之有关的问题，设置该值为 `true` 去使用系统 git 后端

### `installer.max-workers`

类型: `int`

默认值：`number_of_cores + 4 `

使用并行安装程序时设置最大 worker 数。 `number_of_cores` 由 `os.cpu_count()` 确定。 如果这会引发 `NotImplementedError` 异常，则假定 `number_of_cores `为 1。

如果此配置参数设置为大于 `number_of_cores + 4` 的值，则最大 worker 数量仍限制在 `number_of_cores + 4`。

注意：

当 `installer.parallel` 被设定为 `false` 时，配置会被忽略。

### `installer.modern-installation`

类型：`boolean`

默认值：`true`

用一个更现代和快速的方法安装依赖包。

### `installer.no-binary`

类型：`string|boolean`

默认值：`false`

设置这个配置之后，用户可以为所有或特定依赖包配置依赖包分发 (distribution) 格式。

| 配置                     | 描述            |
| ---------------------- | ------------- |
| `:all:` or `true`      | 所有依赖包都禁止二进制格式 |
| `:none:` or `false`    | 所有依赖包都允许二进制格式 |
| `package[,package,..]` | 对某些依赖包禁止二进制格式 |

### `installer.parallel`

类型：`boolean`

默认：`true`

在使用新型安装器（`>=1.1.0`）时，可以使用并行执行的方式进行安装

### `virtualenvs.create`

类型：`boolean`

默认值：`true`

如果还没有一个虚拟环境，就创建一个。如果它被设定为 `false`，Poetry 将不会创建一个新的虚拟环境，如果它在 `{cache-dir}/virtualenvs` 或 `{project-dir}/.venv` 路径下检测到一个虚拟环境。它将往这些环境中安装依赖，否则它会将依赖安装进系统 python 环境。

Notice：如果 Poetry 检测到当前正在一个激活的虚拟环境中运行，它将不再创建一个新的虚拟环境，不管 `virtualenvs.create` 被设置为什么值

### `virtualenvs.in-project`

类型：`boolean`

默认值：`None`

在项目的根目录创建虚拟环境

如果没有显式得设定该值，Poetry 会在 `{cache-dir}/virtualenvs` 或 `{project-dir}/.venv` 路径下默认创建虚拟环境。

如果设定为 `true`，虚拟环境会被创建，并且被期望放在一个名为 `.venv`  的目录名下，该目录在项目的根目录内挂载。

如果设定为 `false`，Poetry 会忽略任何现存的 `.venv` 目录



### 待续

## 仓库

## Refence

[配置](https://python-poetry.org/docs/configuration/)

[仓库](https://python-poetry.org/docs/repositories/)
