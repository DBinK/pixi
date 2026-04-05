# 多环境教程

在本教程中，我们将向你展示如何在单个 Pixi workspace 中使用多个环境。

## 这有什么用处？

在开发 workspace 时，你通常需要不同的工具、库或测试环境。
使用 Pixi，你可以在一个 workspace 中定义多个环境，并在它们之间轻松切换。
开发者通常需要所有可用的工具，而你的测试基础设施可能不需要所有这些工具，而你的生产环境可能需要的更少。
为这些不同用例设置不同的环境可能很麻烦，但使用 Pixi 很容易。

## 词汇表

本教程可能使用一些新术语，这里是快速概述：

#### **Feature**
Feature 定义环境的一部分，但如果没有成为环境的一部分则没有用处。
你可以在一个 workspace 中定义多个 Feature。
一个 Feature 可以包含 `tasks`、`dependencies`、`platforms`、`channels` 和[更多](../reference/pixi_manifest.md#the-feature-table)。
你可以混合多个 Feature 来创建环境。
Feature 通过在 manifest 文件中的表添加 `[feature.<name>.*]` 来定义。
#### **Environment**
环境是 Feature 的集合。
环境实际上可以安装和激活以运行任务。
你可以在一个 workspace 中定义多个环境。
通过在 manifest 文件的 `[environments]` 表中添加它们来定义环境。
#### **Default**
你可以直接填充 `[dependencies]` 而不是指定 `[feature.<name>.dependencies]`。
这些顶级表被添加到 "default" Feature，除非你特别选择退出，否则该 Feature 被添加到每个环境。

## 开始使用

我们将简单地从一个新 workspace 开始，如果你已经有一个 Pixi workspace，你可以跳过此步骤。

```shell
pixi init workspace
cd workspace
pixi add python
```

现在我们有一个包含以下结构的新 Pixi workspace：
```
├── .pixi
│   └── envs
│       └── default
├── pixi.lock
└── pixi.toml
```

注意 `.pixi/envs/default` 目录，这是存储默认环境的位置。
如果未指定环境，Pixi 将创建或使用 `default` 环境。

### 添加 Feature
让我们开始向 workspace 添加一个简单的 `test` Feature。
我们可以通过命令行或编辑 `pixi.toml` 文件来完成。
这里我们将使用命令行，并向 workspace 中的 `test` Feature 添加一个 `pytest` 依赖。
```shell
pixi add --feature test pytest
```
这将把以下内容添加到我们的 `pixi.toml` 文件中：
```toml
--8<-- "docs/source_files/pixi_tomls/multi-environment-simple.toml:test-feat-dep"
```
此表的作用与普通 `dependencies` 表完全相同，但仅在 `test` Feature 是环境的一部分时使用。

### 添加环境
我们将把 `test` 环境添加到我们的 workspace 以添加一些测试工具。
我们可以通过命令行或编辑 `pixi.toml` 文件来完成。
这里我们将使用命令行：
```shell
pixi workspace environment add test --feature test
```
这将把以下内容添加到我们的 `pixi.toml` 文件中：
```toml
--8<-- "docs/source_files/pixi_tomls/multi-environment-simple.toml:test-env"
```

### 运行任务
我们现在可以在新环境中运行任务。
```shell
pixi run --environment test pytest --version
```
这已创建了 test 环境，并在其中运行了 `pytest --version` 命令。
你可以看到环境将被添加到 `.pixi/envs` 目录。
```shell
├── .pixi
│   └── envs
│       ├── default
│       └── test
```
如果你想查看环境，可以使用 `pixi list` 命令。
```shell
pixi list --environment test
```

如果你有始终适合 test 环境的特殊测试命令，你可以将它们添加到 `test` Feature。
```shell
# Adding the 'test' task to the 'test' feature and setting it to run `pytest`
pixi task add test --feature test pytest
```
这将把以下内容添加到我们的 `pixi.toml` 文件中：
```toml
--8<-- "docs/source_files/pixi_tomls/multi-environment-simple.toml:test-tasks"
```
现在你不必在运行测试命令时指定环境。
```shell
pixi run test
```
在这个例子中类似于运行 `pixi run --environment test pytest`

只要只有一个环境有 `test` 任务，这就有效。

## 使用多环境测试包的多个版本
在这个例子中，我们将使用多个环境来针对多个 Python 版本测试一个包。
这是在开发 python 库时的常见用例。
此工作流程可以转换为你希望有多个环境来针对不同依赖设置的任何设置。

在这个例子中，我们假设你已运行了前一个例子中的命令，并有一个带有 `test` 环境的 workspace。
为了允许 python 在新环境中灵活，我们需要将其设置为更灵活的版本，例如 `*`。

```shell
pixi add "python=*"
```

我们将首先设置两个 Feature，`py311` 和 `py312`。
```shell
pixi add --feature py311 python=3.11
pixi add --feature py312 python=3.12
```

我们将把 `test` 和 Python Feature 添加到相应的环境中。
```shell
pixi workspace environment add test-py311 --feature py311 --feature test
pixi workspace environment add test-py312 --feature py312 --feature test
```

这应该导致以下内容添加到 `pixi.toml`：
```toml
--8<-- "docs/source_files/pixi_tomls/multi-environment-py-envs.toml:py-envs"
```

现在我们可以在两个环境中运行测试命令。
```shell
pixi run --environment test-py311 test
pixi run --environment test-py312 test
# Or using the task directly, which will spawn a dialog to select the environment of choice
pixi run test
```

这些现在可以在 CI 中运行来测试单独的环境：
```yaml title=".github/workflows/test.yml"
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      environment: [test-py311, test-py312]
  steps:
  - uses: actions/checkout@v4
  - uses: prefix-dev/setup-pixi@v0
    with:
      environments: ${{ matrix.environment }}
  - run: pixi run -e ${{ matrix.environment }} test
```
更多信息请参阅 GitHub actions [文档](../integration/ci/github_actions.md)。

## 开发、测试、生产环境
假设一个干净的 workspace，所以如果你一直在关注，你可能想要启动一个新的 workspace。
```shell
pixi init production_project
cd production_project
```

和之前一样，我们将从创建多个 Feature 开始。
```shell
pixi add numpy python # default feature
pixi add --feature dev jupyterlab
pixi add --feature test pytest
```

现在我们将添加环境。
为了适应不同的用例，我们将添加一个 `production`、`test` 和 `default` 环境。

- `production` 环境将只有 `default` Feature，因为这是项目运行所需的最低要求。
- `test` 环境将拥有 `test` 和 `default` Feature，因为我们想要测试项目并需要测试工具。
- `default` 环境将拥有 `dev` 和 `test` Feature。

我们使这成为默认环境，因为它将在本地最容易运行，因为它避免了在运行任务时指定环境的需要。

我们还将为环境添加 `solve-group` `prod`，这将确保依赖项被求解，就像它们在同一个环境中一样。
这将导致 `production` 环境具有与 `default` 和 `test` 环境完全相同的依赖项版本。
这样我们可以确保项目在所有环境中以相同的方式运行。

```shell
pixi workspace environment add production --solve-group prod
pixi workspace environment add test --feature test --solve-group prod
# --force is used to overwrite the default environment
pixi workspace environment add default --feature dev --feature test --solve-group prod --force
```

如果我们为环境运行 `pixi list -x`，我们可以看到不同环境具有完全相同的依赖项版本。
```shell
# Default environment
Package     Version  Build               Size       Kind   Source
jupyterlab  4.3.4    pyhd8ed1ab_0        6.9 MiB    conda  jupyterlab
numpy       2.2.1    py313ha4a2180_0     6.2 MiB    conda  numpy
pytest      8.3.4    pyhd8ed1ab_1        253.1 KiB  conda  pytest
python      3.13.1   h4f43103_105_cp313  12.3 MiB   conda  python

Environment: test
Package  Version  Build               Size       Kind   Source
numpy    2.2.1    py313ha4a2180_0     6.2 MiB   conda  numpy
pytest   8.3.4    pyhd8ed1ab_1        253.1 KiB  conda  pytest
python   3.13.1   h4f43103_105_cp313  12.3 MiB   conda  python

Environment: production
Package  Version  Build               Size      Kind   Source
numpy    2.2.1    py313ha4a2180_0     6.2 MiB   conda  numpy
python   3.13.1   h4f43103_105_cp313  12.3 MiB   conda  python
```

### 非默认环境
当你想要一个没有 `default` Feature 的环境时，你可以使用 `--no-default-feature` 标志。
这将导致环境没有 `default` Feature，只有你指定的 Feature。

这的一个常见用例是有一个可以生成文档的环境。

让我们将 `mkdocs` 依赖项添加到 `docs` Feature。
```shell
pixi add --feature docs mkdocs
```

现在我们可以添加没有 default Feature 的 `docs` 环境。
```shell
pixi workspace environment add docs --feature docs --no-default-feature
```

如果我们运行 `pixi list -x -e docs`，我们可以看到它只有 `mkdocs` 依赖项。
```shell
Environment: docs
Package  Version  Build         Size     Kind   Source
mkdocs   1.6.1    pyhd8ed1ab_1  3.4 MiB  conda  mkdocs
```

## 结论

多环境功能非常强大，可以以许多不同的方式使用。
在[参考](../reference/pixi_manifest.md#the-feature-and-environments-tables)和[高级](../workspace/multi_environment.md)部分还有更多值得探索的内容。
如果有任何问题，或者你知道如何改进本教程，请随时在 [GitHub](https://github.com/prefix-dev/pixi) 上联系我们。