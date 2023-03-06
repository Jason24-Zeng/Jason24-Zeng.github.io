---
title: 'Python Project Management - poetry (II)'
date: 2023-03-05 15:27:17
tags: 
- Poetry
- Project Management
- Group
- Publish
categories:
- Coding Language
- Python
cover: /img/python_poetry.webp
top_img: /img/python_poetry.webp
---

## 管理依赖

### 依赖组 groups

Poetry 提供了一种通过 groups 去管理依赖的方法。举个栗子，我们的有些依赖只在测试项目或者构建文档时需要，所以只用给测试项目一个新的依赖组，并在组里加上相关所需依赖即可。

为了声明一个新的依赖组，我们使用 `tool.poetry.group.<group>` 节实现，其中 `<group>` 是我们依赖组的名字，比如 `test`。

```toml
[tool.poetry.group.test]

[tool.poetry.group.test.dependencies]
pytest = "^6.0.0"
pytest-mock = "*"
```

注意如下：

> 所有的依赖都必须在组间互相兼容，因为它们会被 resolve，即使它们在安装阶段不被要求。
> 
> 将依赖组想象成与依赖有关的 labels，它们不必考虑是否依赖会被 resolve 并被默认安装，它们仅仅是一种通过逻辑去管理依赖的简单方法。

> 在 `tool.poetry.dependencies` 中声明的依赖是隐式的 `main` group 中的一部分
> 
> ```toml
> [tool.poetry.dependencies] # main dependency group
> httpx = "*"
> pendulum = "*"
> [tool.poetry.group.test.dependencies]
> pytest = "^6.0.0"
> pytest-mock = "*"
> ```

> 依赖组，除了隐式的 `main` group 以外，必须只能包含我们在开发过程中需要的依赖。只有通过使用 Poetry 安装它们才有可能。
> 
> 如果想要声明一个依赖集合，且这些依赖项在运行期会给项目增加额外的功能，请改用 extras。最终的用户可以使用 pip 安装附加的功能。

> 关于 `dev` 依赖组，我们需要注意如下：
> 
> 定义一个 `dev` 依赖组（poetry 版本需要 > 1.2.0）的可能方法如下
> 
> ```toml
> [tool.poetry.group.dev.dependencies]
> pytest = "^6.0.0"
> pytest-mock = "*"
> ```
> 
> 这个 group notation 从 Poetry 1.2.0 之后被推荐，并且在更早的版本不能使用，要相兼容 Poetry 的老版本，任何在 `dev-dependencies` 中声明的依赖都会被自动添加到 `dev` 依赖组。因此上述的 notations 等价于
> 
> ```toml
> # Poetry pre-1.2.x style, understood by Poetry 1.0–1.2
> [tool.poetry.dev-dependencies]
> pytest = "^6.0.0"
> pytest-mock = "*"
> ```
> 
> 但以上后面的方法会被逐渐弃用，建议使用最新的 notations

#### 可选组 Optional groups

一个依赖组能被声明为可选的。当我们有一组依赖只「在特殊环境被要求时」 或 「在有特殊目的时」有意义。

```toml
[tool.poetry.group.docs]
optional = true

[tool.poetry.group.docs.dependencies]
mkdocs = "*"
```

可选组能被安装通过使用`--with` 选项，它不被默认安装。

```shell
poetry install --with docs
```

Notice: 可选组依赖依然需要和其他依赖一起解决，因此应该特别注意保持它们彼此兼容。

#### 给一个组增加一个依赖

官方推荐使用 `add` 命令去给一个组增加依赖项，这需要加入 `--group` 选项来实现

```shell
poetry add pytest --group test
```

如果组不存在，它会自动被创建。

#### 安装组依赖

默认情况下当我们执行 `poetry install`，所有非可选组的依赖都会被安装。

> 一个项目的默认依赖集合包括隐式的 `main` group（定义的 `tool.poetry.dependencies`），以及所有的不被显式标记被可选项的组

我们可以排除安装一个或多个组的依赖通过使用 `--without` 选项。

```shell
poetry install --without test,docs
```

我们也能操作多安装可选项，通过使用 `with` 选项

```shell
poetry install --with docs
```

Notice：当以上两个选项都被使用的使用，`--without` 的优先级高于 `--with`。比如如下的质量，它只会安装指定的可选 `test` group

```shell
poetry install --with test,docs --without docs
```

最后，在某些情况下，我们可能指向安装某一些指定组的依赖，而不去安装默认依赖集合。为了实现这样的目的，我们可以使用 `--only` 选项

```shell
poetry install --only docs
```

如果我们指向安装项目运行期的依赖，我们可以通过使用 `--only main` 的 notations 来实现

```shell
poetry install --only main
```

如果我们指向安装项目根目录，但不安装其他依赖项，我们可以使用 `--only-root` 选项

```shell
poetry install --no-root
```

#### 从一个组中移除依赖

使用 `remove` 指令配合 `--group` 选项可以支持移除一个特定组的套件。

```shell
poetry remove mkdocs --group docs
```

### 同步依赖

Poetry 支持所谓的依赖同步。依赖同步确保 `poetry.lock` 锁住的依赖是环境中唯一的依赖，它去掉了任何不必要的依赖。

使用方法为在`install` 指令中加入 `--sync` 选项

```shell
poetry install --sync
```

同步选项能与任何依赖组 dependency groups 相关的选项一起使用，从而同步有指定组的环境。需要注意 extras 是分开的。任何没被选来安装的 extras 总是被移除，不管是否有 `--sync`。

```shell
poetry install --without dev --sync
poetry install --with docs --sync
poetry install --only dev
```

`--sync` 选项是用来代替被弃用的 `--remove-untracked` 选项的

### 分层可选组

当我们省略 `--sync` 选项是，我们能安装可选组的任何子集，而无需删除已安装的哪些，这非常有用，比如，在多阶段 Docker 构建中，我们可以在不同的构建阶段多次执行 `poetry install`

## 库 Libraries

这章主要将如何通过 Poetry 让我们的库变为可安装的

### 版本 Versioning

Poetry 要求所有项目的 PEP 440 兼容版本。虽然 Poetry 不强制执行任何发布约束，但他确实鼓励在 PEP 440 范围内使用语义版本控制。这对终端用户好处颇多，并允许它们设定适当的版本约束。

比如， `1.0.0-hotfix.1` 与 PEP 440 不兼容，我们可以选择使用 `1.0.0-post1` 或 `1.0.0.post1` 作为替代。

### 锁文件 `Lock file`

对库而言，我们可以根据自己的喜好提交 `poetry.lock`。这可以帮助团队总是在同一个依赖版本下测试，但是，这个锁文件将不会对其他依赖它的项目有任何影响，它只会影响主项目。

如果我们不想提交锁文件并且我们正在使用 git，可以考虑把它加到 `.gitignore` 中

### 打包

在我们能真正发布我们的库之前，我们需要将其打包

```shell
poetry build
```

这个命令会打包我们的库，它会有两种格式的打包：

- `sdist` 是源格式

- `wheel` 是一种已编译套件

一旦这个完成，我们就可以准确发布我们的库了

### 发布到 PyPI

Poetry 会默认发布库到 PyPI，任何被发布到 PyPI 的东西都能自动通过 Poetry 获得。因为 `pendulum` 在 PyPI 上，我们可以在不用指定任何额外的仓库的情况下依赖它。

如果我们想要风向 `poetry-demo` 到 Python 的社区，我们也会把它发布到 PyPI。操作非常简单

```shell
poetry publish
```

它将打包并且发布这个库到 PyPI，不过前提是账号是一个登记用户，并且已经合理的配置了我们的 凭据 credentials。

> Notice: `publish` 指令不会默认执行 `build`。如果我们想要构建并且发布我们的套件，只用输入 `--build` 选项。

这时，我们的库就已经对任何人可见了。

### 发布到一个私人仓库

优势，我们可能想要保持我们的库 library 私人化，但能被团队成员访问。这种情况，我们就需要使用一个私人仓库 repository。

为了发布到一个私人仓库，我们需要把它添加到全局存储库列表中。

这样之后。我们就能将库发布到这个私人仓库中。

```shell
poetry publish -r my-repository
```

## Reference

[官方文档-依赖章节](https://python-poetry.org/docs/managing-dependencies/)

[PEP 440 – Version Identification and Dependency Specification | peps.python.org](https://peps.python.org/pep-0440/#semantic-versioning)

[Private Repository Example](https://python-poetry.org/docs/repositories/#private-repository-example)
