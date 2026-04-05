# pyproject.toml 支持

我们支持在 pixi 中使用 `pyproject.toml` 作为 manifest 文件。
这允许用户将所有配置保存在一个文件中。
`pyproject.toml` 文件是 Python 项目的标准。
我们不建议将 `pyproject.toml` 文件用于 Python 项目以外的任何类型，`pixi.toml` 更适合其他类型的项目。

## `pyproject.toml` 文件的初始设置

如果你的项目中已经有一个 `pyproject.toml` 文件，你可以在该文件夹中运行 `pixi init`。Pixi 会自动

- 在文件中添加 `[tool.pixi.workspace]` 部分，包含 pixi 所需的 platform 和 channel 信息；
- 将当前项目添加为可编辑的 pypi 依赖项；
- 在 `.gitignore` 和 `.gitattributes` 文件中添加一些默认值。

如果你没有现有的 `pyproject.toml` 文件，你可以在项目文件夹中运行 `pixi init --format pyproject`。在这种情况下，Pixi 将从头开始创建一个包含一些合理默认值的 `pyproject.toml` manifest。

## Python 依赖

`pyproject.toml` 文件支持 `requires_python` 字段。
Pixi 理解该字段并自动将其添加到依赖项中。

这是一个带有 `requires_python` 字段的 `pyproject.toml` 文件示例，它将用作 python 依赖项：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]
```

相当于：

```toml title="equivalent pixi.toml"
[workspace]
name = "my_project"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[dependencies]
python = ">=3.9"
```

## 依赖部分

`pyproject.toml` 文件支持 `dependencies` 字段。
Pixi 理解该字段并自动将依赖项作为 `[pypi-dependencies]` 添加到 workspace。

这是一个带有 `dependencies` 字段的 `pyproject.toml` 文件示例：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = [
    "numpy",
    "pandas",
    "matplotlib",
]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]
```

相当于：

```toml title="equivalent pixi.toml"
[workspace]
name = "my_project"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[pypi-dependencies]
numpy = "*"
pandas = "*"
matplotlib = "*"

[dependencies]
python = ">=3.9"
```

你可以用 conda 依赖项覆盖这些，方法是将它们添加到 `dependencies` 字段：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = [
    "numpy",
    "pandas",
    "matplotlib",
]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[tool.pixi.dependencies]
numpy = "*"
pandas = "*"
matplotlib = "*"
```

这将导致安装 conda 依赖项并忽略 pypi 依赖项。
因为 Pixi 使用 conda 依赖项优先于 pypi 依赖项。

## 可选依赖

如果你的 Python 项目包含可选依赖组，Pixi 会自动将它们解释为相同名称的 [Pixi feature](../reference/pixi_manifest.md#the-feature-table)，以及关联的 `pypi-dependencies`。

你可以手动将它们添加到 Pixi 环境，或使用 `pixi init` 设置 workspace，这将为每个 feature 创建一个环境。
对其他可选依赖组的自引用也会被处理。

例如，假设你有一个项目文件夹，其中 `pyproject.toml` 文件类似：

```toml title="pyproject.toml"
[project]
name = "my_project"
dependencies = ["package1"]

[project.optional-dependencies]
test = ["pytest"]
all = ["package2","my_project[test]"]
```

在该项目文件夹中运行 `pixi init` 会将 `pyproject.toml` 文件转换为：

```toml title="pyproject.toml"
[project]
name = "my_project"
dependencies = ["package1"]

[project.optional-dependencies]
test = ["pytest"]
all = ["package2","my_project[test]"]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64"] # if executed on linux

[tool.pixi.environments]
default = {features = [], solve-group = "default"}
test = {features = ["test"], solve-group = "default"}
all = {features = ["all"], solve-group = "default"}
```

在这个例子中，Pixi 将创建三个环境：

- **default** 带有 'package1' 作为 pypi 依赖
- **test** 带有 'package1' 和 'pytest' 作为 pypi 依赖
- **all** 带有 'package1'、'package2' 和 'pytest' 作为 pypi 依赖

所有环境将被一起解析，如共同的 `solve-group` 所示，并添加到锁文件中。
你可以手动编辑 `[tool.pixi.environments]` 部分以适应你的用例（例如，如果你不需要特定的环境）。

## 依赖组

如果你的 Python 项目包含依赖组，Pixi 会自动将它们解释为相同名称的 [Pixi feature](../reference/pixi_manifest.md#the-feature-table)，以及关联的 `pypi-dependencies`。

你可以手动将它们添加到 Pixi 环境，或使用 `pixi init` 设置 workspace，这将为每个依赖组创建一个环境。

例如，假设你有一个项目文件夹，其中 `pyproject.toml` 文件类似：

```toml
[project]
name = "my_project"
dependencies = ["package1"]

[dependency-groups]
test = ["pytest"]
docs = ["sphinx"]
dev = [{include-group = "test"}, {include-group = "docs"}]
```

在该项目文件夹中运行 `pixi init` 会将 `pyproject.toml` 文件转换为：

```toml
[project]
name = "my_project"
dependencies = ["package1"]

[dependency-groups]
test = ["pytest"]
docs = ["sphinx"]
dev = [{include-group = "test"}, {include-group = "docs"}]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64"] # if executed on linux

[tool.pixi.environments]
default = {features = [], solve-group = "default"}
test = {features = ["test"], solve-group = "default"}
docs = {features = ["docs"], solve-group = "default"}
dev = {features = ["dev"], solve-group = "default"}
```

在这个例子中，pixi 将创建四个环境：

- **default** 带有 'package1' 作为 pypi 依赖
- **test** 带有 'package1' 和 'pytest' 作为 pypi 依赖
- **docs** 带有 'package1', 'sphinx' 作为 pypi 依赖
- **dev** 带有 'package1', 'sphinx' 和 'pytest' 作为 pypi 依赖

所有环境将被一起解析，如共同的 `solve-group` 所示，并添加到锁文件中。
你可以手动编辑 `[tool.pixi.environments]` 部分以适应你的用例（例如，如果你不需要特定的环境）。

## 示例

由于 `pyproject.toml` 文件支持在 `[tool.pixi]` 前缀下的完整 Pixi 规范，示例如下：

```toml title="pyproject.toml"
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = [
    "numpy",
    "pandas",
    "matplotlib",
    "ruff",
]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[tool.pixi.dependencies]
compilers = "*"
cmake = "*"

[tool.pixi.tasks]
start = "python my_project/main.py"
lint = "ruff lint"

[tool.pixi.system-requirements]
cuda = "11.0"

[tool.pixi.feature.test.dependencies]
pytest = "*"

[tool.pixi.feature.test.tasks]
test = "pytest"

[tool.pixi.environments]
test = ["test"]
```

## Build-system 部分

`pyproject.toml` 文件通常包含 `[build-system]` 部分。
如果它作为 pypi 路径依赖项添加，Pixi 将使用此部分来构建和安装项目。

如果 `pyproject.toml` 文件不包含任何 `[build-system]` 部分，Pixi 将回退到 [uv](https://github.com/astral-sh/uv) 的默认值，相当于以下内容：

```toml title="pyproject.toml"
[build-system]
requires = ["setuptools >= 40.8.0"]
build-backend = "setuptools.build_meta:__legacy__"
```

**强烈建议**包含 `[build-system]` 部分。
如果你不确定要使用哪个 [build-backend](https://packaging.python.org/en/latest/tutorials/packaging-projects/#choosing-build-backend)，在 `pyproject.toml` 中包含以下 `[build-system]` 部分是一个很好的起点。
`pixi init --format pyproject` 默认为 `hatchling`。
hatchling 相对于 setuptools 的优势在其[网站](https://hatch.pypa.io/latest/why/#build-backend)上有概述。

```toml title="pyproject.toml"
[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]
```

## 使用 `[tool.uv.sources]` 进行开发依赖

因为 pixi 使用 `uv` 来构建其 `pypi-dependencies`，你可以使用 `tool.uv.sources` 部分来指定来自 main pixi manifest 的任何 pypi-dependencies 的来源。

### 这有什么用处？

当你设置某种类型的 monorepo 并且希望源依赖能够相互引用时，你需要使用 `[tool.uv.sources]` 部分来指定这些依赖的来源。
这是因为 `uv` 处理 PyPI 依赖的解析和任何源依赖的构建。

### 示例

给定一个源码树：

```
.
├── main_project
│   └── pyproject.toml (references a)
├── a
│   └── pyproject.toml (has a dependency on b)
└── b
    └── pyproject.toml
```

具体来说，在 `main_project` 的 `pyproject.toml` 中看起来像这样：

```toml
[tool.pixi.pypi-dependencies]
a = { path = "../a" }
```

然后 `a` 的 `pyproject.toml` 应该包含 `[tool.uv.sources]` 部分。

```toml
[project]
name = "a"
# other fields
dependencies = ["flask", "b"]

[tool.uv.sources]
# Override the default source for flask with main git branch
flask = { git = "github.com/pallets/flask", branch = "main" }
# Reference to b
b = { path = "../b" }
```

有关此部分允许的内容的更多信息，请参阅 [uv 文档](https://docs.astral.sh/uv/concepts/projects/dependencies/#dependency-sources)

!!! note
    主的 `pixi.toml` 或 `pyproject.toml` 由 pixi 直接解析，而不是由 `uv` 处理。
    这意味着你**不能**在主的 `pixi.toml` 或 `pyproject.toml` 中使用 `[tool.uv.sources]` 部分。
    这是一个我们知道的限制，如果你希望支持[这个](https://github.com/prefix-dev/pixi/issues/new/choose)，请随时提出问题。