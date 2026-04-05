# 导入环境教程

在本教程中，我们将向你展示如何将现有环境导入 Pixi workspace。
如果教程中使用的某些词汇对你来说没有意义，你可能首先会从我们的其他教程中获得价值，比如我们的[第一个 workspace 演练](../first_workspace.md)和[多环境 workspace 指南](./multi_environment.md)。

## `pixi import`

在任何 Pixi workspace 中，你可以使用 [`pixi import`](https://pixi.sh/latest/reference/cli/pixi/import/)
从给定文件导入环境。在撰写本文时，我们支持两种导入文件格式：`conda-env` 和 `pypi-txt`。
运行 `pixi import` 而不提供 `format` 将依次尝试每种格式，直到成功，或者如果所有格式都失败则返回错误。

如果你还没有 Pixi workspace，你可以使用 [`pixi init`](https://pixi.sh/latest/reference/cli/pixi/init/) 创建一个。

### `conda-env` 格式

`conda-env` 格式用于 conda 生态系统中的文件（通常称为 `environment.yml`），遵循 [conda 文档中指定的语法](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually)。
假设我们要导入的环境在此文件中指定：

```yaml title="environment.yml"
name: simple-env
channels: ["conda-forge"]
dependencies:
- python
- pip:
  - httpx
```

然后我们可以运行 `pixi import --format=conda-env environment.yml` 将环境导入我们的 workspace。
默认情况下，由于我们的 `environment.yml` 有 `name` 字段，这会创建一个同名的 `feature`（如果该名称的 feature 已存在则使用它），并创建一个包含该 feature 的 `environment`（设置 [`no-default-feature`](https://pixi.sh/latest/reference/pixi_manifest/#the-environments-table)）：

```toml title="pixi.toml"
[feature.simple-env]
channels = ["conda-forge"]

[feature.simple-env.dependencies]
python = "*"

[feature.simple-env.pypi-dependencies]
httpx = "*"

[environments]
simple-env = { features = ["simple-env"], no-default-feature = true }
```

然后可以为导入的环境定义任务，在该环境中运行命令，并在该环境中启动 [`pixi shell`](https://pixi.sh/latest/reference/cli/pixi/shell)——请参阅[入门指南](../getting_started.md)了解有关开始学习这些主题的链接！

对于没有 `name` 字段的文件，或要覆盖默认行为，你可以指定自定义的 `--feature` 和 `--environment` 名称。
这也允许导入到现有 feature 和 environment 中（包括 `default` feature 和 environment）。
例如，给定这个要导入的其他环境文件：

```yaml title="env2.yml"
channels: ["conda-forge"]
dependencies:
- numpy
```

运行 `pixi import --format=conda-env --feature=numpy --environment=simple-env env2.yml` 会将环境导入到一个名为 "numpy" 的新 feature 中，并将该 feature 包含在现有的 `simple-env` 环境（实际上是合并我们两个输入文件中的环境）：

```toml title="pixi.toml"
[feature.simple-env]
channels = ["conda-forge"]

[feature.simple-env.dependencies]
python = "*"

[feature.simple-env.pypi-dependencies]
httpx = "*"

[feature.numpy]
channels = ["conda-forge"]

[feature.numpy.dependencies]
numpy = "*"

[environments]
simple-env = { features = ["simple-env", "numpy"], no-default-feature = true }
```

也可以通过 `--platform` 参数为 feature 指定平台。
例如，`pixi import --format=conda-env --feature=unix --platform=linux-64 --platform=osx-arm64 environment.yml`
将以下内容添加到我们的 workspace manifest：

```toml title="pixi.toml"
[feature.unix]
platforms = ["linux-64", "osx-arm64"]
channels = ["conda-forge"]

[feature.unix.target.linux-64.dependencies]
python = "*"

[feature.unix.target.osx-arm64.dependencies]
python = "*"

[environments]
unix = { features = ["unix"], no-default-feature = true }
```

### `pypi-txt` 格式

`pypi-txt` 格式用于遵循 [`pip` 文档中的需求文件格式规范](https://pip.pypa.io/en/stable/reference/requirements-file-format/) 的 PyPI 生态系统中的文件。

假设我们要导入的环境在此文件中指定：

```yaml title="requirements.txt"
cowpy
array-api-extra>=0.8
```

然后我们可以运行 `pixi import --format=pypi-txt --feature=my-feature1 requirements.txt` 将环境导入我们的 workspace。
必须通过同名的参数指定 `feature` 或 `environment` 名称（或两者）。
如果只提供了其中一个名称，另一个字段使用匹配的名称。
因此，以下行被添加到我们的 workspace manifest：

```toml title="pixi.toml"
[feature.my-feature1.pypi-dependencies]
cowpy = "*"
array-api-extra = ">=0.8"

[environments]
my-feature1 = { features = ["my-feature1"], no-default-feature = true }
```

文件中列出的任何依赖都作为 [`pypi-dependencies`](https://pixi.sh/latest/reference/pixi_manifest/#pypi-dependencies) 添加。
如果给定的 environment 名称尚不存在，将创建一个设置 [`no-default-feature`](https://pixi.sh/latest/reference/pixi_manifest/#the-environments-table) 的环境。

然后可以为该环境定义任务，在该环境中运行命令，并在该环境中启动 [`pixi shell`](https://pixi.sh/latest/reference/cli/pixi/shell)——请参阅[入门指南](../getting_started.md)了解有关开始学习这些主题的链接！

就像 `conda-env` 格式一样，也可以导入到现有 feature/environment（包括 `default` feature/environment），并为 feature 设置特定平台。
有关详细信息，请参阅上一部分。

## `pixi init --import`

也可以将 `pixi init` 和 `pixi import` 的步骤合并为一个，通过 [`pixi init --import`](https://pixi.sh/latest/reference/cli/pixi/init/#arg---import)。
例如，`pixi init --import environment.yml`（使用上面示例中的相同文件）产生一个看起来像这样的 manifest：

```toml title="pixi.toml"
[workspace]
authors = ["Lucas Colley <lucas.colley8@gmail.com>"]
channels = ["conda-forge"]
name = "simple-env"
platforms = ["osx-arm64"]
version = "0.1.0"

[tasks]

[dependencies]
python = "*"

[pypi-dependencies]
httpx = "*"
```

与 `pixi import` 不同，默认情况下这使用 `default` feature 和 environment。
因此，它实现了一个与运行 `pixi init` 和 `pixi import --feature=default environment.yml` 非常相似的 workspace。

一个区别是 `pixi init --import` 默认将从给定的导入文件继承其名称（如果文件指定了 `name` 字段），而不是从其工作目录。

!!! note "支持的格式"
    在撰写本文时，只有 `conda-env` 格式支持 `pixi init --import`。

## 结论

有关更多详细信息，请参阅 [`pixi import`](https://pixi.sh/latest/reference/cli/pixi/import/) 和 [`pixi init --import`](https://pixi.sh/latest/reference/cli/pixi/init/#arg---import) 的 CLI 参考文档。
如果有任何问题，或者你知道如何改进本教程，请随时在 [GitHub](https://github.com/prefix-dev/pixi) 上联系我们。

在撰写本文时，我们有许多扩展导入功能的潜在计划——你可以在 GitHub 上的 `import` [路线图问题](https://github.com/prefix-dev/pixi/issues/4192) 中关注这项工作。