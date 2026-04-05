# 从 uv 切换到 Pixi

本指南帮助你从 uv 过渡到 Pixi。
它比较了两个工具之间的命令和概念，并解释 Pixi 增加了什么：在完整的 conda 生态系统之上管理非 Python 依赖、系统库和多语言项目。

## 为什么选择 Pixi？

uv 是一个快速的 Python 包管理器，但它仅限于 PyPI 生态系统。Pixi 基于 conda，这带来了几个基本优势：

- **包含系统依赖项。** 需要 CUDA、OpenSSL、编译器或 C 库？Conda 包捆绑了它们。使用 uv，你必须通过 `apt`、`brew`、Docker 或手动设置自己安装这些。
- **多语言支持。** 单一 Pixi workspace 可以管理 Python、R、C/C++、Rust、Node.js 等，而 uv 只处理 Python。
- **二进制优先分发。** Conda 包是预编译的，所以你很少需要在机器上安装构建工具链。不必等待源代码构建或调试缺失的 C 头文件。
- **完整的环境建模。** Conda 环境包含一切（解释器、库、头文件、编译器、CLI 工具），全部由求解器管理。使用 uv，你的 Python 环境依赖于系统碰巧提供的任何东西。
- **真正的跨平台锁文件。** Pixi 为单个锁文件中的所有目标平台求解，甚至是你当前不在其上运行的平台。
- **内置任务运行器。** 直接在 manifest 中定义和运行任务，不需要 `Makefile`、`just` 或 shell 脚本。

!!! tip "你仍然可以使用 PyPI 包"
    Pixi 在底层由 uv 提供支持，完全支持 PyPI 包和 conda 包一起使用。
    使用 `pixi add --pypi <package>` 添加 PyPI 依赖，或在使用 `pyproject.toml` 时在 `[project.dependencies]` 中定义它们。
    参阅 [Conda & PyPI](../concepts/conda_pypi.md) 了解两个生态系统如何协同工作。

## 快速了解差异

| 任务                      | uv                                | Pixi                                                                                      |
|---------------------------|-----------------------------------|-------------------------------------------------------------------------------------------|
| 创建项目        | `uv init myproject`               | `pixi init myproject`                                                                     |
| 添加依赖       | `uv add numpy`                    | `pixi add numpy` (conda)或 `pixi add --pypi numpy` (PyPI)                               |
| 移除依赖     | `uv remove numpy`                 | `pixi remove numpy` (conda)或 `pixi remove --pypi numpy` (PyPI)                         |
| 安装/同步        | `uv sync`                         | `pixi install`                                                                            |
| 运行命令         | `uv run python main.py`           | `pixi run python main.py`                                                                 |
| 运行独立脚本 | `uv run script.py` (PEP 723)   | `pixi exec` 通过 [shebang](../advanced/shebang.md)                                        |
| 运行任务            | _(无内置任务运行器)_       | `pixi run my_task`                                                                        |
| 锁定依赖      | `uv lock`                         | `pixi lock`（也在 `pixi add` / `pixi install` 时自动运行）                     |
| 安装 Python         | `uv python install 3.12`          | `pixi add python=3.12`（作为常规依赖管理）                                  |
| 临时工具执行  | `uvx ruff check`                  | `pixi exec ruff check`                                                                    |
| 全局工具安装       | `uv tool install ruff`            | `pixi global install ruff`                                                                |
| 构建包        | `uv build`                        | 通过 [pixi-build backends](../build/getting_started.md) 支持                          |
| 发布包      | `uv publish`                      | 上传到 [prefix.dev channel](../deployment/prefix.md)                                 |
| 导出锁文件      | `uv export`                       | `pixi workspace export conda-environment`                                                 |
| 虚拟环境      | `.venv/` (自动)              | `.pixi/envs/` (自动，支持多环境)                                |
| 缓存管理          | `uv cache clean`                  | `pixi clean cache`                                                                        |
| 更新依赖     | `uv lock --upgrade`               | `pixi update`                                                                             |
| GitHub Actions            | `astral-sh/setup-uv`             | `prefix-dev/setup-pixi`                                                                   |

## 项目配置

uv 使用 `pyproject.toml` 进行项目配置，使用 `uv.toml` 进行工具级设置。
Pixi 支持 `pixi.toml`（其原生格式）和 `pyproject.toml` 进行项目配置，并使用单独的[配置文件](../reference/pixi_configuration.md)进行工具级设置。

=== "uv (pyproject.toml)"

    ```toml
    [project]
    name = "myproject"
    version = "0.1.0"
    requires-python = ">=3.12"
    dependencies = [
        "numpy>=1.26",
        "pandas>=2.0",
    ]

    [dependency-groups]
    dev = ["pytest>=8.0"]
    ```

=== "Pixi (pixi.toml)"

    ```toml
    [workspace]
    name = "myproject"
    channels = ["conda-forge"]
    platforms = ["linux-64", "osx-arm64", "win-64"]

    [dependencies]
    python = ">=3.12"
    numpy = ">=1.26"
    pandas = ">=2.0"

    [feature.test.dependencies]
    pytest = ">=8.0"

    [environments]
    test = ["test"]
    ```

=== "Pixi (pyproject.toml)"

    ```toml
    [project]
    name = "myproject"
    version = "0.1.0"
    requires-python = ">=3.12"
    dependencies = [
        "numpy>=1.26",
        "pandas>=2.0",
    ]

    [dependency-groups]
    test = ["pytest>=8.0"]

    [tool.pixi.workspace]
    channels = ["conda-forge"]
    platforms = ["linux-64", "osx-arm64", "win-64"]

    [tool.pixi.environments]
    test = { features = ["test"], solve-group = "default" }
    ```

使用 `pyproject.toml`，Pixi 将 `[project.dependencies]` 读取为 PyPI 依赖，将 `[tool.pixi.dependencies]` 读取为 conda 依赖。详见 [pyproject.toml 指南](../python/pyproject_toml.md)。

## 概念映射

### Python 版本管理

uv 使用 `uv python install` 单独管理 Python 安装。在 Pixi 中，Python 只是一个普通的包：

```shell
pixi add python=3.12    # 添加 Python 作为 conda 依赖
```

Python 与锁文件中的其他内容一起被版本锁定，因此没有单独的 `.python-version` 文件需要管理。

### 虚拟环境

uv 为每个项目创建一个单一的 `.venv/` 目录。Pixi 在 `.pixi/envs/` 下创建环境，并支持**多个命名环境**同时存在于一个 workspace 中：

```toml title="pixi.toml"
[environments]
default = []
test = ["test"]
docs = ["docs"]
cuda = ["cuda"]
```

每个环境可以有完全不同的（甚至冲突的）依赖项，Pixi 将它们全部并行安装。例如，你可以有一个包含 `numpy 1.x` 的环境和另一个包含 `numpy 2.x` 的环境，两者都准备好使用而无需重新安装任何东西。

uv 可以通过 `tool.uv.conflicts` 在锁文件中分别解析冲突的依赖组，但它仍然使用单一的 `.venv/`，你可以通过 `uv sync --group <name>` 在其中切换。Pixi 环境是独立的目录，因此切换是即时的。

参阅 [Multi Environment](../workspace/multi_environment.md)。

### 依赖组和 extras

uv 使用 [PEP 735 依赖组](https://peps.python.org/pep-0735/) 和可选依赖（extras）来组织依赖项。
Pixi 使用 **features**，可组合的依赖项集合，映射到环境：

| uv                           | Pixi                                                                |
|------------------------------|---------------------------------------------------------------------|
| `[dependency-groups]`        | `[feature.<name>.dependencies]`                                     |
| `[project.optional-dependencies]` | `[feature.<name>.dependencies]` 映射到环境          |
| `uv sync --group dev`       | `pixi install -e dev`                                               |
| `uv sync --all-groups`      | `pixi install --all`                                                |

Features 比依赖组更灵活：它们可以包含 conda 依赖、平台特定的包、系统要求和激活脚本。

### Workspaces

两种工具都支持多包 workspace。uv 在 `pyproject.toml` 中使用 glob 模式定义 workspace 成员：

```toml title="uv pyproject.toml"
[tool.uv.workspace]
members = ["packages/*"]
```

Pixi 采用不同的方法：你直接在 workspace manifest 中将本地包作为路径依赖引用。
任何包含 `[package]` 部分的子目录都可以通过这种方式引入：

```toml title="pixi.toml"
[workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "win-64"]

[dependencies]
my_lib = { path = "packages/my_lib" }
```

两种工具在 workspace 中共享一个锁文件。参阅 [构建多个包](../build/workspace.md)。

### 独立脚本

uv 支持 [PEP 723 内联脚本元数据](https://peps.python.org/pep-0723/) 用于声明自己依赖的独立脚本：

```python title="uv script"
# /// script
# requires-python = ">=3.12"
# dependencies = ["requests"]
# ///
import requests
print(requests.get("https://example.com").status_code)
```

Pixi 通过使用 `pixi exec` 的 [shebang 脚本](../advanced/shebang.md) 有类似的功能，它会创建一个具有指定依赖项的临时环境：

```python title="pixi shebang script"
#!/usr/bin/env -S pixi exec --spec requests --spec python=3.12 -- python
import requests
print(requests.get("https://example.com").status_code)
```

这在 Linux 和 macOS 上有效。更完整的脚本功能正在讨论中，见 [#3751](https://github.com/prefix-dev/pixi/issues/3751)。

### 任务

uv 没有内置的任务运行器。Pixi 有：

```toml title="pixi.toml"
[tasks]
start = "python main.py"
test = "pytest"
lint = "ruff check ."
check = { depends-on = ["lint", "test"] }  # task dependencies
fmt = { cmd = "ruff format .", env = { RUFF_LINE_LENGTH = "120" } }
```

```shell
pixi run check   # runs lint then test
pixi run start
```

任务支持任务间依赖、环境变量、工作目录配置和跨平台命令。参阅 [任务](../workspace/advanced_tasks.md)。

### 临时工具执行（`uvx` vs `pixi exec`）

`uvx`（全称 `uv tool run`）在临时环境中运行工具而不永久安装它。`pixi exec` 做同样的事情：

| uv                              | Pixi                               |
|---------------------------------|------------------------------------|
| `uvx ruff check`               | `pixi exec ruff check`            |
| `uvx --from 'ruff>=0.5' ruff check` | `pixi exec --spec 'ruff>=0.5' ruff check` |
| `uvx --with numpy ruff check`  | `pixi exec --with numpy ruff check` |

### 全局工具（`uv tool` vs `pixi global`）

两种工具都在隔离的环境中全局安装 CLI 工具：

| uv                              | Pixi                               |
|---------------------------------|------------------------------------|
| `uv tool install ruff`         | `pixi global install ruff`         |
| `uv tool list`                 | `pixi global list`                 |
| `uv tool uninstall ruff`       | `pixi global uninstall ruff`       |

因为 Pixi 全局工具来自 conda 生态系统，你也可以安装非 Python 工具：

```shell
pixi global install git bat ripgrep starship
```

参阅 [全局工具](../global_tools/introduction.md)。

### 包索引和 channel

uv 使用 PyPI 作为其默认包索引，支持通过 `[[tool.uv.index]]` 自定义索引。

Pixi 使用 **conda channels**，预编译包的仓库。默认值是 [conda-forge](https://conda-forge.org/)，最大的社区维护 channel：

```toml title="pixi.toml"
[workspace]
channels = ["conda-forge"]
# 添加额外的 channel：
# channels = ["conda-forge", "pytorch", "https://my-company.com/channel"]
```

对于私有包，你可以在 [prefix.dev](https://prefix.dev/) 或 S3 或 JFrog Artifactory 上托管你自己的 channel。参阅 [认证](../deployment/authentication.md)。

### 锁文件

两种工具都生成锁文件以确保可复现性。

| 方面             | uv (`uv.lock`)         | Pixi (`pixi.lock`)                                   |
|--------------------|------------------------|-------------------------------------------------------|
| 格式             | TOML                   | YAML                                                  |
| 跨平台     | 通用解析          | 逐平台求解，存储在一个文件中                |
| 多环境  | 单一解析         | 逐环境解析                            |
| 包类型      | 仅 PyPI              | Conda + PyPI                                          |
| 生成/更新    | `uv lock`              | `pixi lock`（也在 `pixi add` / `pixi install` 时自动） |

参阅 [锁文件](../workspace/lockfile.md)。

### 构建和发布

uv 使用 `uv build`（PEP 517 后端）构建 Python 包，并使用 `uv publish` 发布到 PyPI。

Pixi 通过 [pixi-build](../build/getting_started.md) 构建包，从 Python、C++、Rust、ROS 等生成 conda 包。你可以发布到 [prefix.dev channel](../deployment/prefix.md) 或任何 conda channel。

### CI 与 GitHub Actions

uv 为 GitHub Actions 提供 [`astral-sh/setup-uv`](https://github.com/astral-sh/setup-uv)。
Pixi 有 [`prefix-dev/setup-pixi`](https://github.com/prefix-dev/setup-pixi)，它安装 Pixi、设置缓存并在你的工作流中运行 `pixi install`：

```yaml
- uses: prefix-dev/setup-pixi@v0.8.8
```

参阅 [GitHub Actions](../integration/ci/github_actions.md) 了解更多详情。

### `uv pip` 接口

uv 提供了 `uv pip` 兼容性层（`uv pip install`、`uv pip compile` 等）。

Pixi 没有 pip 兼容性层，它通过 manifest 文件声明性地管理所有依赖项。如果你需要 pip 用于特定用例，你可以将其作为依赖项安装：

```shell
pixi add pip
# 不推荐，首选 pixi add --pypi
pixi run pip install <some-package>
```

!!! warning "首选 `pixi add --pypi`"
    在 Pixi 环境中使用 `pip` 会绕过求解器和锁文件。
    始终首选 `pixi add --pypi <package>` 以保持依赖项可跟踪和可复现。

## 为什么 conda 生态系统很重要

如果你来自 uv，你可能想知道当 PyPI 已经有了你需要的 everything 时，为什么 conda 包很重要。

### 包含系统依赖项

使用 uv，安装 `scipy` 或 `pytorch` 通常需要系统级库（BLAS、LAPACK、CUDA）已经在你的机器上。这导致平台特定的设置说明、仅用于构建依赖的 Docker 容器，或神秘的构建失败。

使用 Pixi，这些系统依赖项是 conda 包，像任何其他依赖项一样由求解器管理：

```shell
# CUDA 运行时、cuDNN 和所有系统库自动解析
pixi add pytorch-gpu
```

### Python 之外的可复现性

`uv.lock` 精确捕获你的 Python 依赖项，但你的项目还依赖于系统的 C 编译器、CUDA 版本、OpenSSL 构建等，这些都没有被跟踪。

`pixi.lock` 捕获**一切**：Python 解释器、系统库、编译器和 CLI 工具。当同事克隆你的项目并运行 `pixi install` 时，他们获得完全相同的环境。没有"在我机器上工作"的惊喜。

### 不需要 Docker 来隔离环境

使用 uv 的常见模式是使用 Docker 来获得具有正确系统依赖项的可复现环境。Pixi 环境实现相同的隔离而不需要容器：不需要 root 权限，比容器镜像小得多，创建即时，并且通过锁文件提供相同的可复现性保证。

### 前向兼容

Conda 包针对最旧的支持的系统基线构建，因此它们也可以在新版 OS 上工作。今天的锁文件明年仍能正确安装在 OS 版本上。

要更深入地了解 conda 和 PyPI 生态系统之间的差异，请参阅 [Conda != PyPI](https://conda.org/blog/conda-is-not-pypi) 博客文章系列。

## 迁移项目

要将现有的 uv 项目迁移到 Pixi，首先在项目目录中初始化 Pixi：

=== "pixi.toml"

    ```shell
    pixi init --format pixi
    ```

    这会在你的 `pyproject.toml` 旁边创建一个 `pixi.toml`。

=== "pyproject.toml"

    ```shell
    pixi init --format pyproject
    ```

    这会将 `[tool.pixi.workspace]` 部分添加到你现有的 `pyproject.toml`，保持你的 PyPI 依赖项不变。

1. **尽可能使用 conda-forge 包而不是 PyPI：**

    Conda 包捆绑系统库和预编译二进制文件，所以 `pixi add numpy` 给你包含 BLAS、LAPACK 和所有内容的 numpy。仅对 conda-forge 上不可用的包使用 `pixi add --pypi <package>`。如果 conda-forge 缺少你需要的包，考虑[自己添加](https://github.com/pavelzw/skill-forge/blob/main/recipes/conda-forge/SKILL.md)。

2. **设置任务来替换你的脚本：**

    ```shell
    pixi task add test "pytest"
    pixi task add lint "ruff check ."
    pixi task add serve "python -m http.server"
    ```

5. **运行你的项目：**

    ```shell
    pixi run test
    pixi run python main.py
    ```

一旦一切在 `pixi.lock` 中正常工作，你可以删除 `uv.lock`。