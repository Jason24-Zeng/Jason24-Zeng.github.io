---
title: Python Project Management - poetry (III)
date: 2023-03-05 22:20:44
tags: 
- Poetry
- Project Management
- Commands
categories:
- Coding Language
- Python
passwd: 0418
cover: /img/python_poetry.webp
top_img: /img/python_poetry.webp
---

## 命令

通过前两篇文章，我们已经学会使用命令行接口做一些事了，而这一章则囊括了所有的相关命令。如果想要从命令行中得到帮助，可以简单的调用 `poetry` 去查看完整的命令列表，然后对其中任何命令后面加上 `--help` 就能获得更多有用信息。

### 全局选项

- `--verbose(-v|vv|vvv)` 增加消息的详细程度，"-v" 为正常输出，"-vv" 为更详细得输出，"-vvv" 则是用于 debug

- `--help(-h)` 显式帮助信息

- `--quiet(-q)` 不输出任何信息

- `--ansi` 强制 ANSI 输出

- `--no-ansi` 禁用 ANSI 输出

- `--version(-V)` 显式应用版本

- `--no-interaction(-n)` 不询问任何交互问题

- `--no-plugins` 禁用 plugins

- `--no-cache` 禁用 Poetry 源 cache

- `--directory=DIRECTORY(-C)` Poetry 指令的工作目录，默认为当前工作目录

### new

这个指令会帮我们启动一个新的 Python 项目，并创建一个适合大部分项目的目录结构

```shell
poetry new my-package
```

创建的结构如下：

```shell
my-package
├── pyproject.toml
├── README.md
├── my_package
│   └── __init__.py
└── tests
    └── __init__.py
```

如果我们想要将我们的项目命名成一个区别与主目录的名字，我们可以加上 `--name` 选项

```shell
poetry new my-folder --name my-package
```

如果我们想要有一个 src 目录，可以使用 `--src` 选项 

```shell
poetry new --src my-package
```

这个指令会创建一下的目录结构

```shell
my-package
├── pyproject.toml
├── README.md
├── src
│   └── my_package
│       └── __init__.py
└── tests
    └── __init__.py
```

另外 `--name` 选项是足够只能的，它能检测出 namespace 套件，并且为我们创建要求的结构

```shell
poetry new --src --name my.package my-package
```

它会创建如下结构：

```shell
my-package
├── pyproject.toml
├── README.md
├── src
│   └── my
│       └── package
│           └── __init__.py
└── tests
    └── __init__.py
```

#### 选项

- `--name`: 设定最终 package 的名字

- `--src`：使用项目的 src 布局

- `--readme`：指定 readme 文件扩展，默认是 `md`，如果我们想要发布到 PyPI，需要牢记 [写一个 PyPI 友好 README 的推荐](https://packaging.python.org/en/latest/guides/making-a-pypi-friendly-readme/)

### init

这个命令帮我们交互式得创建一个 `pyproject.toml` 文件，我们需要的是提供一些关于我们套件的基本信息。

它会交互式得要求我们填写一些信息，同时使用一些只能的默认值

```shell
poetry init
```

#### 选项

- `--name`：package 的名字

- `--description`：package的描述

- `--author`：package的作者

- `--python`: 可兼容的 Python 版本

- `--dependency`：要求的一些依赖库，同时需要版本约束。格式如 `foo:1.0.0`

- `--dev-dependency`：开发要求的一些依赖库。

### install

`install` 指令会读取 当前项目的`pyproject.toml` 文件，解析出依赖项，并且安装它们。

```shell
poetry install
```

如果有一个 `poetry.lock` 文件在当前目录中，它会使用 `poetry.lock` 文件中使用的完全相同的版本依赖，而不是解析它们。这确保了每个使用该库的人都能获得完全相同版本的抵赖。

如果没有一个 `poetry.lock` 文件，Poetry 会在依赖解析后创建一个。

如果我们想要在安装时不包含一个或多个依赖组，我们可以选择使用 `--without` 选项。

```shell
poetry install --without test,docs
```

我们也可以使用 `--with`选项选择安装可选依赖组的依赖

```shell
poetry install --with test,docs
```

当然也可以只安装指定的依赖组，这时我们需要使用 `--only` 选项

```shell
poetry install --only test,docs
```

只安装项目自身，而不安装依赖，使用 `--only-root` flag

```shell
poetry install --only-root
```

如果我们想要同步环境，并且确保它匹配锁文件，我们可以使用 `--sync` 选项

```shell
poetry install --sync
```

`--sync` 选项能合与依赖组有关的选项一起使用 

```shell
poetry install --without dev --sync
poetry install --with docs --sync
poetry install --only dev
```

我们还可以指定我们想要安装的 extras，这时我们需要传递 `-E|--extras` 选项。而传递 `--all-extras` 会安装项目中所有定义好的 extras

```shell
poetry install --extras "mysql pgsql"
poetry install -E mysql -E pgsql
poetry install --all-extras
```

extras 对 `--sync` 不敏感，任何没有之规定的 extras 都会被移除

```shell
poetry install --extras "A B"  # C 被移除
```

默认的，每次我们运行 `install`选项，`poetry` 会安装项目的依赖包：

```shell
$ poetry install
Installing dependencies from lock file

No dependencies to install or update

  - Installing <your-package-name> (x.x.x)
```

如果我们想要跳过安装，可以使用 `--no-root` 选项

```shell
poetry install --no-root
```

默认情况下，安装阶段 `poetry` 不会编译 Python 源文件成字节码。这会加速安装过程，但是首次执行会有一点耗时，因为那时候 Python 会自动编译源代码成字节码。如果想要在安装阶段就做这样的编译操作，可以使用`--compile` 选项

```shell
poetry install --compile
```

#### 选项

- `--without` ：将要忽略的依赖组

- `--with`：将要包含的可选依赖组

- `--only`：将只包含的依赖组

- `--only-root` ：只安装根项目，不包含所有的依赖

- `--sync` ：同步环境中锁定的 package 和 指定的组

- `--no-root`：不安装根 package，即我们的项目

- `--dry-run`：输出操作，但不执行任何操作

- `--extras(-E)`：安装的特征，允许多值。

- `--all-extras`：安装所有的 extra 特征，与 `--extras` 冲突

- `--compile`：编译 Python 源代码成字节码

需要注意，当指定了 `--only` 时，`--with` 和 `--without` 选项都会被忽略。

### update

为了得到最新版本的依赖，并且更新 `poetry.lock` 文件，我们可以使用 `update` 指令

```shell
poetry update
```

该指令会解析项目的所有依赖，并且往 `poetry.lock` 文件写入真实版本

如果我们指向更新一些 package，而不是所有的，我们可以执行如下操作

```shell
poetry update requests toml
```

不过需要注意：这里的更新不会超过 `pyproject.toml`文件内指定的版本限制。也就是说， 举个例子如`poetry update foo` ，如果 `foo` 被要求是 `~2.3` 后者 `2.3`，而此时有版本 `2.4` ， `foo` 并不会被更新。想要更新 `foo`，我们必须先更新限制，比如更新成 `^2.3`。想要更新限制，可以使用 add 指令。

#### 选项

- `--without` ：要忽略的依赖组

- `--with`：要包含的可选依赖组

- `--only`：只包含的依赖组

- `--dry-run`：输出操作，但不执行任何操作

- `--lock`：不执行安装，只更新锁文件。

### add
