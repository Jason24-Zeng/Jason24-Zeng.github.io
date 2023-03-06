---
title: 'Python Project Management - poetry (I)'
date: 2023-02-23 23:06:55
tags: 
- Poetry
- Project Management
categories:
- Coding Language
- Python
cover: /img/python_poetry.webp
top_img: /img/python_poetry.webp
---

## 简介

在 Python 项目中，Poetry 起到了依赖管理和打包的工作。它允许我们去宣称项目所依赖的库，并且它可以帮我们管理这些库，主要包括安装，更新和卸载等。另外 Poetry 提供一个锁文件，它能确保不重复安装，并且为发布 build 我们的项目。

### 系统要求

Poetry 需要安装高于 Python3.7 的版本，并且它是多平台的，其目标立志于在 Linux，macOs 和 Windows 系统中都工作得一样好。

### 安装

有许多安装 Poetry 的方式，比如官方安装器，手动安装，以及 `pipx` 安装，这里不过多介绍，一般的安装方式即：

```bash
pipx install poetry
# 执行哪个版本对应的 pip 安装 poetry
python3.9 -m pip install poetry
```

### Tab 自动补全功能

Poetry 支持为 Bash, Fish 和 Zsh 产生自动补全脚本，可以通过输入命令 `poetry help completions` 获得详细信息，不过要点如下

#### Bash

```bash
# 自动 loaded 
poetry completions bash >> ~/.bash_completion
# lazy loaded
poetry completions bash > ${XDG_DATA_HOME:-~/.local/share}/bash-completion/completions/poetry
```

#### Zsh

```shell
poetry completions zsh > ~/.zfunc/_poetry
```

如果 `~/.zshrc` 文件中不存在下面行，我们需要添加

```shell
fpath+=~/.zfunc
autoload -Uz compinit && compinit
```

#### On My Zsh

```shell
mkdir $ZSH_CUSTOM/plugins/poetry
poetry completions zsh > $ZSH_CUSTOM/plugins/poetry/_poetry
```

同时，我们需要在 `~/.zshrc` 文件中将 `poetry` 加入到 `plugins` 数组中

```shell
plugins(
    poetry
    ...
)
```

## 基础使用

为基础使用介绍，我们将安装 `pendulm`，一个 `datetime` 库。

### 项目配置

#### 新项目创建

首先，创造一个新项目，比如 `poetry-test`

```bash
poetry new poetry-test
# Created package poetry_test in poetry-test
```

它会创建一个 `poetry-test` 目录，并且该目录包括一下内容。

```shell
> tree poetry-test
-----------------
poetry-test
├── README.md
├── poetry_test
│   └── __init__.py
├── pyproject.toml
└── tests
    └── __init__.py
```

这里需要着重关注一下 `pyproject.toml` ，它对于 poetry 进行项目管理至关重要，它会编排整个项目以及依赖，在刚开始初始化的时候，它是这样的：

```yml
[tool.poetry]
name = "poetry-test"
version = "0.1.0"
description = ""
authors = ["Jason24-Zeng <z5zeng@eng.ucsd.edu>"]
readme = "README.md"
packages = [{include = "poetry_test"}]

[tool.poetry.dependencies]
python = "^3.9"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

需要特别注意这时候的 python 版本，它需要被明确得指明，因为这个是你项目的依赖，需要特别注意是否该版本满足当前项目的要求，后面的所有依赖项都需要声称支持该版本。

Poetry 假设系统套件（package）中包含一个套件，该套件与 `tool.poetry.name` 同名，位于项目的根部。如果不满足这个情况，需要填充 `tool.poetry.packages` 来制定套件及其位置。

相似的，创痛的 `MANIFEST.in` 文件被 `tool.poetry.readme` ，`tool.poetry.include` 以及 `tool.poetry.excluede` section 取代。`tool.poetry.exclude` 被 `.gitignore` 隐式填充。

什么意思呢，在详细查看了 `pyproject section` 后，发现是说，如果一个 VCS(version control system) 被用于项目， `exclude` 字段将使用 VCS 的忽略设置，比如 git 的 `.gitignore` 作为种子。而如果没有，那么在 `exclude` 字段会只出一个文件路径的集合，这些文件在 package built 的时候不会被包括在内。形式如下

```python
exclude = ["my_package/excluded.py"]
```

#### 初始化一个已有项目

除构造新项目外，Poetry 还可以被用来初始化一个已经存在的目录。为了在目录 `pre-existing-project` 目录下交互式得创建一个 `pyproject.toml` 文件，可以操作如下：

```shell
cd pre-existing-project
poetry init
```

#### 指定依赖 Specifying Dependencies

如果我们想要给我们的项目增加依赖，我们需要在 `tool.poetry.dependencies` 节中制定它们，如下，我们可以引入一个套件名到版本限制的映射：

```toml
[tool.poetry.dependencies]
pendulum = "^2.1"
```

Poetry 会使用这些信息去套件仓库中搜索正确的文件集合，这些仓库可以来自于 `tool.poetry.source` 节的注册，也可能来自于 默认的`PyPI`。

另外， 除了直接手动的修改 `pyproject.toml` 文件以外，我们也可以使用 `add` 命令去添加依赖：

```shell
poetry add pendulum
```

该指令会自动找到合适的版本限制，并且安装套件和其子依赖。

Poetry 支持许多依赖指定的语法，详细可参考：[Dependency specification](https://python-poetry.org/docs/dependency-specification/)

### 使用虚拟环境

默认情况下，Poetry 会创建一个虚拟环境到 `{cache-dir}/vritualenvs`。我们可以通过修改 Poetry 的配置项来改变 `cache-dir` 的值。另外，我们也可以使用 `virtualenvs.in-project` 配置变量在项目目录下创建虚拟环境。而在虚拟环境中，我们有一下多种方式去运行指令。

> 外部虚拟环境管理
> 
> Poetry 将检测并尊重已存在的外部激活的虚拟环境，这是一种强大的机制，只在替代 Poetry 内置的简化环境管理。
> 
> 要利用这点，我们只用通过喜欢的方法或工具激活虚拟环境，再运行任何期望操纵环境的 Poetry 命令

#### 使用 `poetry run`

如果要运行脚本，可以简单的使用 `poetry run python your_script.py` 。相似的，如果有命令行工具比如 `pytest` 或 `black` 我们也可以使用 `poetry run pytest` 去运行。

#### 激活虚拟环境

最简单的激活虚拟环境的方法是创建一个嵌套 shell，可通过执行 `poetry shell` 创建。如果想要停用环境并且退出，则使用这个新的 shell，并输入 `exit`。如果指向停用环境，但不退出 shell，可以使用 `deactivate`。

可替代地，为了避免创建一个新的 shell，我们可以人为得激活虚拟环境，只需要执行 `source {path_to_venv}/bin/activate`。为了获得虚拟环境的路径，可以执行 `poetry env info --path`。为此，我们可以合并成一行的指令 `source $(poetry env info --path)/bin/activate`。要停用这个虚拟环境只用简单使用 `deactivate`。

另外，在 stack overflow 中还提及到一种方法，帮助找到已经 activated 的虚拟环境路径，并激活它，指令如下 

```shell
source "$( poetry env list --full-path | grep Activated | cut -d' ' -f1 )/bin/activate"
```

### 版本限制

在上面的例子中，我们想要一个版本限制为 `^2,1` 的 `pendulum` 套件。这意味着版本需要控制在大于等于 2.1.0 但小于 3.0.0（`>=2.1.0``<3.0.0`）。详细可参考：[Dependency specification](https://python-poetry.org/docs/dependency-specification/)

### 安装依赖

想要安装项目定义好的依赖，只用运行 `install` 指令：

```shell
poetry install
```

当我们运行这个指令的时候，会发生下面两种情况中的一种

#### 没有 `poetry.lock` 情况下安装

如果我们在之前从来没有运行过这个 command，并且没有 `poetry.lock` 文件在项目目录内，Poetry 会简单地解析列在 `pyproject.toml` 文件中的所有依赖并且安装这些依赖套件的最新版本。

当 Poetry 结束安装，他会把所有的 package 和他们真实下载的版本写到 `poetry.lock` 文件，并以此锁定项目到使用这些指定版本。我们应该 commit `poetry.lock` 文件到项目仓库，这样才能让所有工作在项目中的人都被锁定到同一个依赖的版本。

#### 有 `poetry.lock` 情况下安装

这是第二种场景，如果已经有一个 `poetry.lock` 和 `pyproject.toml` 文件时，如果我们运行 `poetry install` ，它意味着要么我们之前运行过 `poetry install` 指令，要么项目的其他人运行过 `install` 命令，并且 commit `poetry.lock` 文件到项目中。

无论是上面哪种情况，在一个 `poetry.lock` 文件存在时运行 `install`，都会安装我们列在 `pyproject.toml` 的所有依赖，但 `Poetry` 使用列在 `poetry.lock` 中的真实版本，它确保 package 的版本和工作在这个项目的其他人保持一致。最终，我们会拥有所有 `pyproject.toml` 文件要求的依赖，但他们可能不是非常最新可用的版本。这是被设计好的，而不是问题，它能确保项目不会因为依赖中不可预期的改变而被打断。

#### 提交 `poetry.lock` 文件给版本控制。

提交这个文件到版本控制是重要的，因为它会让其他任何配置这个项目的人使用「和我们使用的完全相同版本的」依赖。我们的连续继承服务器（Continuous Integration Server），生产机器，团队的其他开发者，每件事和每个人都在同样的依赖下运行，这减少了「仅影响部署的某些部分」的 bug 的可能性。即使我们独立开发，6 个月后，当我们需要重新安装项目，我们也能自信安装的依赖依然能工作，即使发布的依赖到那时会有许多全新的版本。

#### 只安装依赖

现在的项目在默认情况下是在可编辑模式下安装的，如果我们指向安装依赖，可以执行 `install` 指令，同时加上 `--no-root` 的 flag:

```shell
poetry install --no-root
```

### 更新依赖到最新版本

上面提到， `poetry.lock` 文件防止我们自动获得依赖的最新版本。为了更新最新的版本，使用 `update` 命令，他将根据 `pyproject.toml` 文件获取最新的匹配版本，并且更新 `poetry.lock` 文件到新版本。这等同于删除 `poetry.lock` ，并重新运行 `install`

> Poetry 会显示一个 Warning 如果执行一个安装指令时，`poetry.lock` 和 `pyproject.toml` 不同步

## Reference

[Poetry 官方文档](https://python-poetry.org/docs/#enable-tab-completion-for-bash-fish-or-zsh)

[poetry Virtual environment already activated](https://stackoverflow.com/questions/60580332/poetry-virtual-environment-already-activated)
