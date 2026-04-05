# Python 教程

在本教程中，我们将向你展示如何使用 Pixi 创建一个简单的 Python 项目。
我们将展示 Pixi 提供的一些功能，这些功能目前不在 `pdm`、`poetry` 等工具中。

## 这有什么用处？

Pixi 基于 conda 生态系统构建，允许你创建包含所有依赖项的 Python 环境。
当你使用多个 Python 解释器以及 C 和 C++ 库的绑定时，这特别有用。
例如，来自 PyPI 的 GDAL 没有二进制 C 依赖项，但 conda 包有。
另一方面，有些包只能通过 PyPI 获得，`pixi` 也可以为你安装。
集两家之长，让我们开始吧！

## `pixi.toml` 和 `pyproject.toml`

我们支持两种 manifest 格式：`pyproject.toml` 和 `pixi.toml`。
在本教程中，我们将使用 `pyproject.toml` 格式，因为它是 Python 项目最常见的格式。

## 开始使用

让我们开始创建一个使用 `pyproject.toml` 文件的新项目。

```shell
pixi init pixi-py --format pyproject
```

这将创建一个包含以下结构的项目目录：

```shell
pixi-py
├── pyproject.toml
└── src
    └── pixi_py
        └── __init__.py
```

项目的 `pyproject.toml` 如下所示：

```toml
[project]
dependencies = []
name = "pixi-py"
requires-python = ">= 3.11"
version = "0.1.0"

[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["osx-arm64"]

[tool.pixi.pypi-dependencies]
pixi_py = { path = ".", editable = true }

[tool.pixi.tasks]
```

此项目使用 src-layout，但 Pixi 同时支持 [flat-layout 和 src-layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/#src-layout-vs-flat-layout)。

### `pyproject.toml` 中有什么？

好的，让我们看看添加了哪些部分，以及如何修改 `pyproject.toml`。

首先添加到 `pyproject.toml` 文件的条目如下：

```toml
# Main pixi entry
[tool.pixi.workspace]
channels = ["conda-forge"]
# This is your machine platform by default
platforms = ["osx-arm64"]
```

`channels` 和 `platforms` 被添加到 `[tool.pixi.workspace]` 部分。
像 `conda-forge` 这样的 channel 管理类似于 PyPI 的包，但允许不同语言使用不同的包。
`platforms` 关键字决定 workspace 支持哪些平台。

`pixi_py` 包本身作为可编辑依赖项添加。
这意味着该包以可编辑模式安装，因此你可以更改包并在环境中看到更改，而无需重新安装。

```toml
# Editable installs
[tool.pixi.pypi-dependencies]
pixi-py = { path = ".", editable = true }
```

在 pixi 中，与其他包管理器不同，这明确写在 `pyproject.toml` 文件中。
主要原因是你可以选择将这个包包含在哪个环境中。

### 同时管理 Conda 和 PyPI 依赖

我们的项目通常依赖于其他包。

```shell
cd pixi-py # 进入项目目录
pixi add black
```

这将把 `black` 包作为 Conda 包添加到 `pyproject.toml` 文件中。
这将导致以下内容添加到 `pyproject.toml`：

```toml
[tool.pixi.dependencies]
black = ">=25.1.0,<26"  # (1)!
```

1. 或者是 [conda-forge](https://prefix.dev/channels/conda-forge/packages/black) channel 上可用的最新版本。

但我们也可以严格指定要使用的版本。
```shell
pixi add black=25
```
结果：

```toml
[tool.pixi.dependencies]
black = "25.*"
```

有时有些包在 conda channel 上不可用，但发布在 PyPI 上。

```shell
pixi add black --pypi
```
这会导致在 `pyproject.toml` 的 `dependencies` 键中添加：

```toml
dependencies = ["black"]
```

使用 `pypi-dependencies` 时，你可以利用其他包提供的 `optional-dependencies` 作为 extra。
例如，`flask` 提供 `async` 依赖选项，可以使用 `--pypi` 关键字添加：

```shell
pixi add "flask[async]==3.1.0" --pypi
```
这会将 `dependencies` 条目更新为

```toml
dependencies = ["black", "flask[async]==3.1.0"]
```

??? note "Extras in `pixi.toml`"
    本教程重点介绍 `pyproject.toml` 的使用，但如果你好奇，在安装包含可选依赖的 PyPI 包后，`pixi.toml` 将包含以下条目：
    ```toml
    [pypi-dependencies]
    flask = { version = "==3.1.0", extras = ["async"] }
    ```

### 安装：`pixi install`

Pixi 在其中运行内容之前始终确保环境与 `pyproject.toml` 文件保持最新。
如果你想手动执行，可以运行：

```shell
pixi install
```

现在我们在 workspace 根目录中有一个名为 `.pixi` 的新目录。
该环境是一个 Conda 环境，所有 Conda 和 PyPI 依赖都已安装到其中。

该环境始终是 `pixi.lock` 文件的结果，该文件由 `pyproject.toml` 文件生成。
此文件包含跨平台安装到环境中的依赖项的确切版本。

## 环境中有什麼？

使用 `pixi list`，你可以查看环境中的内容，这本质上是锁文件（`pixi.lock`）的更好视图：

```shell
Package          Version     Build               Size       Kind   Source
asgiref          3.8.1                           68.5 KiB   pypi   asgiref-3.8.1-py3-none-any.whl
black            24.10.0     py313h8f79df9_0     388.7 KiB  conda  black
blinker          1.9.0                           23.9 KiB   pypi   blinker-1.9.0-py3-none-any.whl
bzip2            1.0.8       h99b78c6_7          120 KiB    conda  bzip2
ca-certificates  2024.12.14  hf0a4a13_0          153.4 KiB  conda  ca-certificates
click            8.1.8       pyh707e725_0        82.7 KiB   conda  click
flask            3.1.0                           335.9 KiB  pypi   flask-3.1.0-py3-none-any.whl
itsdangerous     2.2.0                           45.8 KiB   pypi   itsdangerous-2.2.0-py3-none-any.whl
jinja2           3.1.5                           484.8 KiB   pypi   jinja2-3.1.5-py3-none-any.whl
libexpat         2.6.4       h286801f_0          63.2 KiB   conda  libexpat
libffi           3.4.2       h3422bc3_5          38.1 KiB   conda  libffi
liblzma          5.6.3       h39f12f2_1          96.8 KiB   conda  liblzma
libmpdec         4.0.0       h99b78c6_0          67.6 KiB   conda  libmpdec
libsqlite        3.48.0      h3f77e49_1          832.8 KiB  conda  libsqlite
libzlib          1.3.1       h8359307_2          45.3 KiB   conda  libzlib
markupsafe       3.0.2                           73 KiB     pypi   markupsafe-3.0.2-cp313-cp313-macosx_11_0_arm64.whl
mypy_extensions  1.0.0       pyha770c72_1        10.6 KiB   conda  mypy_extensions
ncurses          6.5         h5e97a16_3          778.3 KiB  conda  ncurses
openssl          3.4.0       h81ee809_1          2.8 MiB    conda  openssl
packaging        24.2        pyhd8ed1ab_2        58.8 KiB   conda  packaging
pathspec         0.12.1      pyhd8ed1ab_1        40.1 KiB   conda  pathspec
pixi_py          0.1.0                                      pypi    (editable)
platformdirs     4.3.6       pyhd8ed1ab_1        20 KiB     conda  platformdirs
python           3.13.1      h4f43103_105_cp313  12.3 MiB   conda  python
python_abi       3.13        5_cp313             6.2 KiB    conda  python_abi
readline         8.2         h92ec313_1          244.5 KiB  conda  readline
tk               8.6.13      h5083fa2_1          3 MiB      conda  tk
tzdata           2025a       h78e105d_0          120 KiB    conda  tzdata
werkzeug         3.1.3                           743 KiB    pypi   werkzeug-3.1.3-py3-none-any.whl
```
在这里，你可以看到列出了不同的 conda 和 Pypi 包。
如你所见，我们正在开发的 `pixi_py` 包以可编辑模式安装。
Pixi 中的每个环境都是隔离的，但重用了从中央缓存目录硬链接的文件。
这意味着你可以有多个具有相同包的环境，但只在磁盘上存储一次文件。

??? question "为什么环境中有 Python 解释器？"
    Python 解释器也被安装到环境中。
    这是因为 Python 解释器版本是从 `pyproject.toml` 文件中的 `requires-python` 字段读取的。
    这用于确定要安装到环境中的 Python 版本。
    这样，Pixi 自动为你管理/引导 Python 解释器，因此不再需要 `brew`、`apt` 或其他系统安装步骤。

??? question "如何使用自由线程解释器？"
    如果你想使用自由线程 Python 解释器，可以使用以下命令添加 `python-freethreading` 依赖：
    ```
    pixi add python-freethreading
    ```
    这确保将自由线程版本的 Python 安装到环境中。
    这可能不适用于尚非线程安全的其他包。
    你可以在[这里](https://docs.python.org/3/howto/free-threading-python.html)阅读有关自由线程 Python 的更多信息。

### 多环境
Pixi 还可以创建多个环境，这可以与 `pyproject.toml` 文件中的 `dependency-groups` 功能很好地配合。

让我们添加一个依赖组，Pixi 称之为 `feature`，命名为 `test`。
并将 `pytest` 包添加到这个组。

```shell
pixi add --pypi --feature test pytest
```

这将导致该包按照 [PEP 735](https://peps.python.org/pep-0735/) 添加到 `dependency-groups` 中。

```toml
[dependency-groups]
test = ["pytest"]
```

将 `dependency-groups` 添加到 `pyproject.toml` 后，Pixi 将它们视为一个 [`feature`](../reference/pixi_manifest.md/#the-feature-and-environments-tables)，可以包含 `dependencies`、`tasks`、`channels` 等的集合。


```shell
pixi workspace environment add default --solve-group default --force
pixi workspace environment add test --feature test --solve-group default
```

结果：

```toml
[tool.pixi.environments]
default = { solve-group = "default" }
test = { features = ["test"], solve-group = "default" }
```

??? note "Solve Groups"
    Solve groups 是一种将依赖项分组的方式。
    当你有多个共享相同依赖项的环境时，这很有用。
    例如，也许 `pytest` 是一个影响 `default` 环境依赖项的依赖项。
    通过将它们放在同一个 solve group 中，你可以确保 `test` 和 `default` 中的版本完全相同。

不指定环境名称时，Pixi 默认为 `default` 环境。
如果你想安装或运行 `test` 环境，请在命令中添加 `--environment test`。

```shell
pixi install --environment test
pixi run --environment test pytest
```

## 让代码运行

让我们为 `pixi_py` 包添加一些代码。
我们将在 `src/pixi_py/__init__.py` 文件中添加一个新函数：

```python
from rich import print

def hello():
    return "Hello, [bold magenta]World[/bold magenta]!", ":vampire:"

def say_hello():
    print(*hello())
```

现在从 PyPI 添加 `rich` 依赖
```shell
pixi add --pypi rich
```

让我们通过运行来检查这是否有效：

```shell
pixi run python -c 'import pixi_py; pixi_py.say_hello()'
```
应该输出：

```shell
Hello, World! 🧛
```

??? note "慢？"
    第一次可能有点慢，因为 Pixi 安装了项目，但第二次会很快。

Pixi 运行自安装的 Python 解释器。
然后，我们导入以可编辑模式安装的 `pixi_py` 包。
代码调用我们刚刚添加的 `say_hello` 函数。
成功了！太棒了！

## 测试这个代码

好的，让我们为这个函数添加一个测试。
让我们在项目根目录添加一个 `tests/test_me.py` 文件。

给我们以下项目结构：

```shell
.
├── pixi.lock
├── src
│   └── pixi_py
│       └── __init__.py
├── pyproject.toml
└── tests/test_me.py
```

```python
from pixi_py import hello

def test_pixi_py():
    assert hello() == ("Hello, [bold magenta]World[/bold magenta]!", ":vampire:")
```

让我们添加一个运行测试的简单任务。

```shell
pixi task add --feature test test "pytest"
```

所以 Pixi 有一个任务系统来简化运行命令。
类似于 `npm scripts` 或你在 `Justfile` 中指定的内容。

!!! tip "Pixi 任务"
    任务是 Pixi 的一个强大功能，在跨平台 shell 中运行。
    你可以进行缓存、依赖等。
    在[任务](../workspace/advanced_tasks.md)部分阅读更多内容。

```shell
pixi run test
```
结果如下输出：
```shell
✨ Pixi task (test): pytest .
================================================================================================= test session starts =================================================================================================
platform darwin -- Python 3.12.2, pytest-8.1.1, pluggy-1.4.0
rootdir: /private/tmp/pixi-py
configfile: pyproject.toml
collected 1 item

test_me.py .                                                                                                                                                                                                    [100%]

================================================================================================= 1 passed in 0.00s =================================================================================================
```

??? question "为什么我不必指定环境？"
    `test` 任务被添加到 `test` feature/环境中。
    当你运行 `test` 任务时，Pixi 会自动切换到 `test` 环境，
    因为它是与 `test` feature 关联的唯一环境。

太棒了！看起来它可以工作！

### Test vs Default 环境

让我们比较 test 和 default 环境的输出。
我们添加 `--explicit` 标志来显示环境中的显式依赖。

```shell
pixi list --explicit --environment test
# vs. default environment
pixi list --explicit
```

我们看到 `test` 环境有：

```shell
package          version       build               size       kind   source
...
pytest           8.1.1                             1.1 mib    pypi   pytest-8.1.1-py3-none-any.whl
...
```

但是，default 环境缺少 `pytest` 包。
这样，你就可以微调环境，只包含需要的包。
例如，你还可以有一个包含 `pytest` 和 `ruff` 的 `dev` 环境，
但你可以从 `prod` 环境省略这些。有一个 [docker](https://github.com/prefix-dev/pixi/tree/main/examples/docker) 示例展示了如何设置最小的 `prod` 环境并从中复制。

## 用 conda 包替换 PyPI 包

最后，Pixi 提供了让 `pypi` 包依赖 `conda` 包的能力。
让我们用以下命令确认：
```shell
pixi list pygments
```
请注意它是作为 `pypi` 包安装的：
```shell
Package          Version       Build               Size       Kind   Source
pygments         2.17.2                            4.1 MiB    pypi   pygments-2.17.2-py3-none-any.http.whl
```

这是 `rich` 包的依赖项。
你可以通过运行以下命令看到：
```shell
pixi tree --invert pygments
```

让我们明确地将 `pygments` 添加到 `pyproject.toml` 文件中。

```shell
pixi add pygments
```

这会将以下内容添加到 `pyproject.toml` 文件中：

```toml
[tool.pixi.dependencies]
pygments = "=2.19.1,<3"
```

现在我们可以看到 `pygments` 包是作为 conda 包安装的。

```shell
pixi list pygments
```
现在结果：
```shell
Package   Version  Build         Size       Kind   Source
pygments  2.19.1   pyhd8ed1ab_0  867.8 KiB  conda  pygments
```

这样，PyPI 依赖项和 conda 依赖项可以混合匹配以无缝互操作。

```shell
pixi run python -c 'import pixi_py; pixi_py.say_hello()'
```

仍然有效！

## 结论

在本教程中，你已经看到使用 `pyproject.toml` 管理 Pixi 依赖项和环境有多么容易。
我们还探索了如何在同一 workspace 中无缝协同使用 PyPI 和 conda 依赖项，并安装可选依赖项来管理 Python 包。

希望这为管理你的 Python 项目提供了一种灵活而强大的方式，并为进一步探索 Pixi 奠定了基础。

感谢阅读！快乐编程 🚀

有任何问题？请随时联系我们或在 [X](https://twitter.com/prefix_dev) 上分享本教程，[加入我们的 Discord](https://discord.gg/kKV8ZxyzY4)，发送我们[电子邮件](mailto:hi@prefix.dev)或关注我们的 [GitHub](https://github.com/prefix-dev)。