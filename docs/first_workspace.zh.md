# 创建 Pixi workspace

Pixi 最大的优势在于其创建可复现、强大且灵活的 workspace 的能力。
workspace 存在于系统上的一个目录中，是环境的集合，可用于在该目录中开发一个或多个项目。让我们来看看创建简单 Pixi workspace 的常见步骤。

## 创建 Pixi workspace

要创建新的 Pixi workspace，可以使用 `pixi init` 命令：

```shell
pixi init my_workspace
```

此命令创建一个名为 `my_workspace` 的新目录，包含以下结构：

```shell
my_workspace
├── .gitattributes
├── .gitignore
└── pixi.toml
```

`pixi.toml` 文件是 Pixi workspace 的 manifest。
它包含关于 workspace 的所有信息，如 channel、platform、dependencies、task 等。

`pixi init` 创建的文件是一个最小的 manifest，内容如下：

```toml title="pixi.toml"
[workspace]
authors = ["Jane Doe <jane.doe@example.com>"]
channels = ["conda-forge"]
name = "my_workspace"
platforms = ["osx-arm64"]
version = "0.1.0"

[tasks]

[dependencies]
```

??? tip "想要 manifest 文件的自动补全吗？"
    由于 `pixi.toml` 有 JSON schema，可以使用 VSCode 等 IDE 编辑字段并获得自动补全。
    安装 [Even Better TOML VSCode 扩展](https://marketplace.visualstudio.com/items?itemName=tamasfe.even-better-toml) 以获得最佳体验。
    或使用 PyCharm 内置的 schema 支持。

## 管理依赖

创建 workspace 后，你可以开始添加依赖。
Pixi 使用 `pixi add` 命令向 workspace 添加依赖。
默认情况下，此命令会将 [**conda**](https://prefix.dev/blog/what-is-a-conda-package) 依赖添加到 `pixi.toml`，解析依赖，写入[锁文件](./workspace/lockfile.md)，并将包安装到环境中。例如，让我们添加 `numpy` 和 `pytest` 到 workspace。

```shell
pixi add numpy pytest
```
这会添加以下内容：

```toml title="pixi.toml"
[dependencies]
numpy = ">=2.2.6,<3"
pytest = ">=8.3.5,<9"
```

你也可以指定要添加的依赖版本。

```shell
pixi add numpy==2.2.6 pytest==8.3.5
```

### PyPI 依赖

Pixi 通常使用 `conda` 包作为依赖，但你也可以从 [PyPI](https://pypi.org) 添加依赖。
Pixi 会确保不会尝试从两个来源安装同一个包，并避免它们之间的冲突。

如果你想将它们添加到 workspace，可以使用 `--pypi` 标志：

```shell
pixi add --pypi httpx
```
这会将 PyPI 的 `httpx` 包添加到 workspace：

```toml title="pixi.toml"
[pypi-dependencies]
httpx = ">=0.28.1,<0.29"
```

要了解更多关于 `conda` 和 PyPI 之间的区别，请参阅 [Conda & PyPI 概念文档](./concepts/conda_pypi.md)。

## 锁文件

Pixi 在解析依赖时总是会创建锁文件。
此文件将包含 workspace 依赖项（及其依赖项）的所有精确版本。
这会产生可复现的环境，你可以与他人共享，并用于测试和部署。

锁文件称为 `pixi.lock`，创建在 workspace 的根目录。
要了解更多关于锁文件的信息，请参阅[详细的锁文件文档](./workspace/lockfile.md)。

```yaml title="pixi.lock"
version: 6
environments:
  default:
    channels:
    - url: https://prefix.dev/conda-forge/
    indexes:
    - https://pypi.org/simple
    packages:
      osx-arm64:
      - conda: https://prefix.dev/conda-forge/osx-arm64/bzip2-1.0.8-h99b78c6_7.conda
      - pypi: ...
packages:
- conda: https://prefix.dev/conda-forge/osx-arm64/bzip2-1.0.8-h99b78c6_7.conda
  sha256:adfa71f158cbd872a36394c56c3568e6034aa55c623634b37a4836bd036e6b91
  md5:fc6948412dbbbe9a4c9ddbbcfe0a79ab
  depends:
  - __osx >=11.0
  license: bzip2-1.0.6
  license_family: BSD
  size: 122909
  timestamp: 1720974522888
- pypi: ...
```

## 管理任务

Pixi 内置了跨平台的任务运行器，允许你在 manifest 中定义任务。
将任务视为你可能想在项目开发过程中重复多次的命令（或命令链）（例如，运行测试）。

这是与他人共享任务并确保在不同机器上运行相同任务的好方法。
任务在 `pixi.toml` 文件的 `[tasks]` 部分定义。

你可以运行 `pixi task add` 命令添加一个任务到 workspace。

```shell
pixi task add hello "echo Hello, World!"
```
这会将以下内容添加到 `pixi.toml` 文件：

```toml title="pixi.toml"
[tasks]
hello = "echo Hello, World!"
```
然后你可以使用 `pixi run` 命令运行该任务：

```shell
pixi run hello
```

这将在 workspace 的默认环境中执行命令 `echo Hello, World!`。

??? tip "想使用更强大的功能吗？"
    任务可以更强大，例如：

    ```toml
    [tasks.name-of-powerful-task]
    cmd = "echo This task can do much more! Like have {{ arguments }} and {{ "minijinja" | capitalize }} templates."

    # List of tasks that must be run before this one.
    depends-on = ["other-task"]

    # Working directory relative to the root of the workspace
    cwd = "current/working/directory"

    # List of arguments for the task
    args = [{ arg = "arguments", default = "default arguments" }]

    # Run the command if the input files have changed
    input = ["src"]
    # Run the command if the output files are missing
    output = ["output.txt"]

    # Set environment variables for the task
    env = { MY_ENV_VAR = "value" }
    ```
    更多关于任务的信息请参阅文档的[任务](./workspace/advanced_tasks.md)部分。

## 环境

Pixi 始终为你的 workspace 创建一个环境（"default" 环境），
其中包含你的 `dependencies`，你的任务也在其中运行。
你也可以在一个 workspace 中包含[多个环境](./workspace/multi_environment.md)。
这些环境位于 workspace 根目录的 `.pixi/envs` 目录中。

使用这些环境非常简单，只需运行 `pixi run` 或 `pixi shell` 命令。
`pixi run` 会在环境中执行剩余输入作为命令（如果输入匹配定义的任务名称，则作为任务），
而 `pixi shell` 会在环境中生成新的 shell 会话。
这两个命令都会"激活"环境——了解更多：
[环境激活文档](./workspace/environment.md#activation)。

```shell
pixi run python -VV
# or:
pixi shell
python -VV
exit
```

想了解你刚才所做事情背后的概念吗——packages、
channels、platforms？继续阅读
[The Conda Ecosystem](conda_ecosystem.md)。