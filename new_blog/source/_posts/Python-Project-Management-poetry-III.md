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

`add` 指令增加要求的依赖包到 `pyproject.toml` 文件，并且安装它们。如果我们不指定一个版本限制，Poetry 会在可选依赖包版本里选择一个合适的。

```shell
poetry add requests pendulum
```

我们也可以指定一个限制：

```shell
# 允许 >=2.0.5, <3.0.0 的版本
poetry add pendulum@^2.0.5

# 允许 >2.0.5, <2.1.0 的版本
poetry add pendulum@~2.0.5

# 允许 >= 2.0.5 版本，没有上限
poetry add "pendulum>=2.0.5"

# 只允许 2.0.5 版本
poetry add pendulum==2.0.5
```

如果我们尝试增加一个已经存在的依赖包，我们会得到一个报错。不过，如果我们制定一个如上的限制，依赖会被更新到指定限制的版本。

如果我们想要得到已经存在依赖的最新版本，我们可以使用特殊的限制 `latest`

```shell
poetry add pendulum@latest
```

我们也能添加 `git` 依赖

```shell
poetry add git+https://github.com/sdispater/pendulum.git
```

或者用 ssh 取代 https

```shell
poetry add git+ssh://git@github.com/sdispater/pendulum.git

# or alternatively:
poetry add git+ssh://git@github.com:sdispater/pendulum.git
```

如果我们想要切刀特定分支，tag或版本，可以通过使用 add 指定它

```shell
poetry add git+https://github.com/sdispater/pendulum.git#develop
poetry add git+https://github.com/sdispater/pendulum.git#2.0.5

# or using SSH instead:
poetry add git+ssh://github.com/sdispater/pendulum.git#develop
poetry add git+ssh://github.com/sdispater/pendulum.git#2.0.5
```

或者引用一个子目录

```shell
poetry add git+https://github.com/myorg/mypackage_with_subdirs.git@main#subdirectory=subdir
```

另外，你也可以添加一个本地目录或文件

```shell
poetry add ./my-package/
poetry add ../my-package/dist/my-package-0.1.0.tar.gz
poetry add ../my-package/dist/my_package-0.1.0.whl
```

如果我们想要依赖能在可编辑模式下被安装，我们可以使用 `--editable` 选项

```shell
poetry add --editable ./my-package/
poetry add --editable git+ssh://github.com/sdispater/pendulum.git#develop
```

可替代的，我们也可以在`pyproject.toml` 文件中声明它。这意味着在本地目录的改变会直接反映到环境中

```toml
[tool.poetry.dependencies]
my-package = {path = "../my/path", develop = true}
```

如果我们想要安装的依赖包提供 extras，我们能在添加该依赖包时指定 extras：

```shell
poetry add "requests[security,socks]"
poetry add "requests[security,socks]~=2.22.0"
poetry add "git+https://github.com/pallets/flask.git@1.1.1[dotenv,dev]"
```

如果我们想要给一个指定依赖组添加依赖包，我们能使用 `--group(-G)` 操作

```shell
poetry add mkdorc --group docs
```

#### 选项

- `--group(-G)` 要将依赖包添加到的依赖组名

- `--dev(-D)` 添加依赖包到开发依赖，目前已经被弃用，使用 `-G dev` 代替。

- `--editable(-e)` 增加 vcs 依赖或者路径依赖，并且令这些依赖为可编辑的。

- `--extras(-E)` 添加的依赖需要激活的 extras，允许多值

- `--optional` 增加一个可选依赖

- `--python` 要想使用依赖必须安装的 python 版本

- `--platform` 依赖必须要安装到的平台

- `--source` 使用来安装依赖的源名字

- `--allow-prereleases` 允许提前发布

- `--dry-run` 输出操作，但不执行任何操作

- `--lock` 不实现安装，值更新锁文件



### remove

`remove` 操作会从当前已安装依赖包中移除一个依赖包

```shell
poetry remove pendulum
```

如果我们想要为一个指定依赖组移除一个依赖包，我们可以使用 `-G` 选项

```shell
poetry remove mkdocs --group docs
```

#### 选项

- `--group`  移除依赖项的组

- `--dev(-D)` 从开发依赖移除依赖包

- `--dry-run` 输出操作，但不执行任何操作



### show

为了列出所有可选依赖包，我们可以使用 `show` 命令

```shell
poetry show
```

如果想要看一个指定依赖包的细节，我们能传入依赖包名字

```shell
poetry show pendulum

name        : pendulum
version     : 1.4.2
description : Python datetimes made easy

dependencies
 - python-dateutil >=2.6.1
 - tzlocal >=1.4
 - pytzdata >=2017.2.2

required by
 - calendar >=1.4.0
```

#### 选项

- `--without` 要忽略的依赖组

- `--why` 当站址整个列表，或者对一个单一依赖包使用 `-tree` 时，展示为什么一个依赖包会被 include。

- `--with` 要包含的可选依赖组

- `--only` 要仅包含的依赖组

- `--no-dev` 不用列出 dev dependencies

- `--tree` 将依赖按照树状图的格式罗列

- `--latest(-l)` 展现最近的版本

- `--outdated(-o)` 展现最新的版本，但进适用于过时的依赖包

- `--all(-a)` 展现所有的依赖包，即使有些与当前系统不兼容

### build

`build` 指令会去构建源档案和轮子档案。

```shell
poetry build
```

需要注意，有时，只有纯 python 轮子被支持

#### 选项

- `--format(-f)` 限制格式，要么是 `wheel` ，要么是 `sdist`

### publish

这个指令发布已经通过 `build` 指令构建好的依赖包，到远程仓库。

如果这是第一次提交，那么在上传之前，该指令会自动登记这个 package。

```shell
poetry publish
```

如果我们加上 `--build` 选项，他也能构建依赖包。

#### 选项

- `--repository(-r)`  依赖包要被登记到的仓库，默认是 `pypi`，它必须和 `config` 指令设定的其中一个仓库匹配

- `--username(-u)`  访问存储库的用户名

- `--password(-p)`  访问存储库的密码

- `--cert` 访问存储库的证书颁发机构

- `--client-cert`  用于访问存储库的客户端证书

- `--build` 在发布之前构建依赖

- `--dry-run`  除了上传依赖包以外，实现所有的操作

- `--skip-existing` 忽略在仓库中已存在文件的错误



### config

`config` 指令允许我们编辑 poetry 配置设置和仓库

```shell
poetry config ==list
```

#### 使用

```shell
poetry config [options] [setting-key] [setting-value1] ... [setting-valueN]
```

`setting-key` 是一个配置可选名而 `setting-value1` 则是一个配置值。

#### 选项

- `--unset` 移除 被 `setting-key` 命名的配置元素

- `--list` 展示当前配置变量的列表

- `--local` 设置或者得到 对一个项目特殊的 配置（一个配置局部路径 `poetry.toml`）

### run

`run` 指令在项目的虚拟环境中执行给定的命令

```shell
poetry run python -V
```

它也能执行 `pyproject.toml` 中定义的脚本之一。

所以，如果我们已经定义一个如下的脚本

```toml
[tool.poetry.scripts]
my-script = "module:main"
```

我们能如下执行脚本：

```shell
poetry run my-script
```

需要注意该指令没有可选项

### shell

`shell` 指令根据 `$SHELL` 环境变量在虚拟环境中生成一个 shell，如果还不存在一个虚拟环境，则该指令会先创建一个。

```shell
poetry shell
```

注意这个指令会创建一个新的 shell 并且激活虚拟环境。因此，我们应该使用 `exit` 来正确退出 shell 和虚拟环境，而不是停用。

### check

`check` 指令会检验 `pyproject.toml` 文件结构的有效性，如果有任何问题，它会返回一个细节报告。该指令也并用做 pre-commit hook。

```shell
poetry check
```

### search

该指令会在远程索引上搜索依赖包

```shell
poetry search requests pendulum
```

### lock

该指令锁住在 `pyproject.toml` 中制定的依赖，但是不会安装。

```shell
poetry lock
```

#### 选项

- `--check` 验证 `poetry.lock` 和 `pyproject.toml` 一致

- `--no-update` 不更新锁定的版本，只更新 lock 文件



### version

该指令展示项目当前的版本，或者提升项目的版本，并将新版本写回 `pyproject.toml`（如果提供了有效的提升规则的话）

新版本需要是一个有效地 [PEP 440](https://peps.python.org/pep-0440/) 字符串或者有效地提升规则：`patch`,`minor`,`major`,`prepatch`,`preminor`,`premajor`,`prerelease`。

| RULE       | BEFORE  | AFTER   |
|:----------:|:-------:|:-------:|
| major      | 1.3.0   | 2.0.0   |
| minor      | 2.1.4   | 2.2.0   |
| patch      | 4.1.1   | 4.1.2   |
| premajor   | 1.0.2   | 2.0.0a0 |
| preminor   | 1.0.2   | 1.1.0a0 |
| prepatch   | 1.0.2   | 1.0.3a0 |
| prerelease | 1.0.2   | 1.0.3a0 |
| prerelease | 1.0.3a0 | 1.0.3a1 |
| prerelease | 1.0.3b0 | 1.0.3b1 |

#### 选项

- `--short(-s)` 只输出版本号

- `--dry-run` 不更新 `pyproject.toml` 文件

### export

这个指令导出 lock 文件到其他格式

```shell
poetry export -f requirements.txt --output requirements.txt
```

该指令被 [Export Poetry Plugin](https://github.com/python-poetry/poetry-plugin-export) 提供，并且也可以被用作一个 pre-commit hook。

在不指定任何选项是，这个指令只会包含定义在 `main` 依赖组里的项目以来，即 `tool.poetry.dependencies` 中的，这和 `install` 选项不同。

#### 选项

- `--format(-f)` 导出的格式，默认为 `requirements.txt`。当前，只支持 `constraints.txt` 和 `requirement.txt`

- `--output(-o)` 输出文件的名字，如果不填，它会输出到标准输出中

- `--dev` 包含开发环境依赖，目前已被弃用，使用 `--with dev` 代替

- `--extras(-E)` 包含的额外依赖集合

- `--without` 忽略的依赖组

- `--with` 包含的可选依赖组

- `--only` 仅会包含的依赖组

- `--without-hashes` 从导出的文件去除 hashes

- `--without-url` 从导出的文件中去除源仓库 urls

- `--with-credentials` 包括额外索引的凭据

### env

`env` 环境指令重组子命令以与特定项目关联的 virtualenvs（虚拟环境） 交互。详细可以查询  [Managing environments](https://python-poetry.org/docs/managing-environments/) 的相关信息

### cache

`cache` 指令会重组子命令，以与 Poetry 的缓存进行交互

#### cache list

`cache list` 命令会列出 Poetry 的所有可用 缓存

```shell
poetry cache list
```

#### cache clear

`cache clear` 指令会从一个缓存的仓库中移除依赖包

比如，为了清理 `pypi` 仓库内依赖包的所有缓存，可以执行：

```shell
poetry cache clear pypi --all
```

如果只想从缓存中移除一个指定的依赖包，我们必须指定缓存入口，方式如 `cache:package:version`

```shell
poetry cache clear pypi:requests:2.24.0
```

### source

`source` 命名空间重组子命令，并以此为一个 Poetry 项目管理仓库源

#### source add

`source add ` 指令为项目增加源码配置。比如，为了增加 `pypi-test` 源码，我们可以执行

```shell
poetry source add pypi-test https://test.pypi.org/simple/
```

注意，我们不能使用 `pypi` 作为名字，因为它以备用来作为默认的 `PyPI` 源

##### 选项

- `--default` 设置这个源为默认，取消 PyPI 的默认

- `--secondary` 将该源设为次要源

我们不能将一个源既设定成 default 又设定成 secondary

#### source show

`source show` 指令展现有关项目所有已配置源的信息

```shell
poetry source show
```

可选的，我们能只展现指定的某个或某几个源

```shell
poetry source show pypi-test
```

该指令只会展示通过 `pyproject.toml` 中配置的源，并且不会包括 PyPI

#### source remove

`source remove` 指令会从 `pyproject.toml` 中移除一个配置的源

```shell
poetry source remove pypi-test
```

### about

`about` 指令可以展示关于 Poetry 的全局信息，包括当前版本和 `poetry-core` 的版本

```shell
poetry about
```

### help

`help` 指令显式全局的帮助。或者一个特殊指令的帮助。

想要展示全局的帮助选项：

```shell
poetry help
```

为了展示对某个特定指令的帮助，比如 `show`，输出指令如下

```shell
poetry help show
```

`--help` 指令也能被传给任何指令以得到特定指令的帮助，比如：

```shell
poetry show --help
```

### list

`list` 指令展现所有可用的 Poetry 指令

```shell
poetry list
```

### self

`self` 命名空间重组子命令去管理自身的 Poetry 安装。使用这些指令会在我们的配置目录下创建要求的 `pyproject.toml` 和 `poetry.lock` 文件

#### self add

`self add` 命令安装 Poetry 插件，并且让他们在运行期可用。另外，他也能被使用来更新 `Poetry` 自身的依赖或者想运行环境输入额外的依赖包。

> `self add` 指令和 `add` 指令的效果几乎一样，唯一的不同是，其管理的依赖包是在 Poetry 运行环境生效的。
> 
> 被 `self add` 支持的依赖包指定格式和 `add` 指令完全一致

例如，为了安装 `poetry-plugin-export` ，我们可以执行 

```shell
poetry self add poetry-plugin-export
```

为了安装最新的 `poetry-core` 版本，我们可以运行

```shell
poetry self add poetry-core@latest
```

为了安装一个钥匙提供器 `artifacts-keyring`，我们可以执行

```shell
poetry self add artifacts-keyring
```

##### 选项

- `--editable(-e)` 以可编辑的权限增加 vcs/path 依赖

- `--extras(-E)` 为了依赖需要激活的特征

- `--allow-prereleases` 允许提前发布

- `--source` 使用来安装依赖包的源名

- `--dry-run` 输出执行操作但是不执行任何东西，它会隐式得打开 `-verbose`

#### self update

`self update` 指令会在它当前的运行期环境中更新 Poetry 版本

```shell
poetry self update
```

##### 选项

- `--preview` 允许预发布版本的安装

- `--dry-run` 输出执行操作但是不指定任何东西，它会隐式得打开 `-verbose`

#### self lock

`self lock` 指令会阅读该 Poetry 安装的系统 `pyproject.toml` 文件。系统依赖被锁定在对应的 `poetry.lock` 文件中 

```shell
poetry self lock
```

##### 选项

- `--check` 检验 `poetry.lock` 与 `pyproject.toml` 一致

- `--no-update` 不安装锁定的版本，只更新锁文件

#### self show

`self show` 指令和 `show` 指令表现相似，但是是在 Poetry 运行期环境内生效。它会列举在 Poetry 安装环境中安装的所有依赖包

为了只展示通过 `self add` 增加的额外依赖包及其依赖，我们可以使用 `self show --addons`

```shell
poetry self show
```

##### 选项

- `--addons` 只列举 addon 附加安装的依赖包

- `--tree` 以树结构形式列举依赖

- `--latest(-l)` 展示最新版本

- `--outdated(-o)` 只为已经过时的依赖包展示最新版本

#### self show plugins

`self show plugins` 指令会列举当前所有安装的插件

```shell
poetry self show plugins
```

#### self remove

`self remove` 指令移除一个安装的 addon 附加依赖包

```shell 
poetry self remove poetry-plugin-export
```

##### 选项

- `--dry-run` 输出执行操作但是不指定任何东西，它会隐式得打开 `-verbose`

#### self install

`self install` 指令确保所有被指定的额外依赖包都已经被安装到当前运行环境中。

```shell
poetry self install --sync
```

##### 选项

- `--sync` 与锁定的依赖包和指定的组进行同步

- `--dry-run`  输出执行操作但是不指定任何东西，它会隐式得打开 `-verbose`



## Reference

[Poetry Command Instruction](https://python-poetry.org/docs/cli/#options-13)
