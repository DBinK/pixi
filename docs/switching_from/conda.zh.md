# 从 Conda/Mamba 切换到 Pixi

欢迎阅读本指南，帮助你从 `conda` 或 `mamba` 过渡到 `pixi`。
本文档比较了这些工具之间的关键命令和概念，突出了 `pixi` 独特的环境和包管理方法。
使用 `pixi`，你将体验到基于 workspace 的工作流程，增强你的开发过程，并允许轻松分享你的工作。

## 为什么选择 Pixi？

`Pixi` 建立在 conda 生态系统的基础上，引入了以 workspace 为中心的方法，而不是仅仅关注环境。
这种向 workspace 的转变提供了一种更有组织、更高效的方式来管理依赖项和运行代码，适合现代开发实践。

## 关键差异一览

| 任务                        | Conda/Mamba                                       | Pixi                                                                      |
|-----------------------------|---------------------------------------------------|---------------------------------------------------------------------------|
| 安装                | 需要安装器                             | 下载并添加到路径（参阅[安装](../index.md)）                |
| 创建环境     | `conda create -n myenv -c conda-forge python=3.8` | `pixi init myenv` 后跟 `pixi add python=3.8`                       |
| 激活环境   | `conda activate myenv`                            | `pixi shell` 在 workspace 目录内                                 |
| 停用环境 | `conda deactivate`                                | `exit` 退出 `pixi shell`                                              |
| 运行任务              | `conda run -n myenv python my_program.py`         | `pixi run python my_program.py`（参阅[run](../reference/cli/pixi/run.md)） |
| 安装包        | `conda install numpy`                             | `pixi add numpy`                                                          |
| 卸载包      | `conda remove numpy`                              | `pixi remove numpy`                                                       |


!!! warn "没有 `base` 环境"
    Conda 有一个 base 环境，这是你在新 shell 启动时的默认环境。
    **Pixi 没有 base 环境**。它要求你在 workspace 或全局安装你需要的工具。
    使用 `pixi global install bat` 会在全局环境中安装 `bat`，这与 conda 中的 `base` 环境不同。

??? tip "在当前 shell 中激活 Pixi 环境"
    对于一些高级用例，你可以在当前 shell 中激活环境。
    这使用 `pixi shell-hook`，它打印激活脚本，可以不带 `pixi` 本身使用。
    ```shell
    ~/myenv > eval "$(pixi shell-hook)"
    ```

## 环境 vs Workspace

`Conda` 和 `mamba` 专注于管理环境，而 `pixi` 强调 workspace。
在 `pixi` 中，workspace 是一个文件夹，包含一个 [manifest](../reference/pixi_manifest.md)（`pixi.toml`/`pyproject.toml`）文件描述 workspace，一个描述确切依赖项的 `pixi.lock` 锁文件，以及一个包含环境的 `.pixi` 文件夹。

这种以 workspace 为中心的方法允许轻松共享和协作，因为 workspace 文件夹包含重新创建环境所需的所有信息。
它在一个 workspace 中管理多个平台和多个环境，并允许在它们之间轻松切换。（参阅[多环境](../workspace/multi_environment.md)）

## 全局环境

`conda` 在一个全局位置安装所有环境。
当这对你的文件系统很重要时，你可以使用 pixi 的 [detached-environments](../reference/pixi_configuration.md#detached-environments) 功能。

```shell
pixi config set detached-environments true
# 或特定位置
pixi config set detached-environments /path/to/envs
```

这将使环境的安装转到同一个文件夹。

`pixi` 确实有 `pixi global` 命令来安装机器上的工具。（参阅[全局](../reference/cli/pixi/global.md)）
这不是 `conda` 的替代品，但与 [`pipx`](https://pipx.pypa.io/stable/) 和 [`condax`](https://mariusvniekerk.github.io/condax/) 相同。
它为给定要求创建一个隔离的环境，并将二进制文件安装到全局路径。

```shell
pixi global install bat
bat pixi.toml
```

!!! warn "永远不要用 `pixi global` 安装 `pip`"
    使用 `pixi global` 的安装会获得自己的隔离环境。
    用 `pixi global install pip` 会在自己的隔离环境中安装 `pip`。
    使用该 `pip` 二进制文件会在 `pip` 环境中安装包，使其无法从任何地方访问，因为你无法激活它。

## 自动切换

你可以将 `environment.yml` 文件导入 Pixi workspace——参阅我们的[导入教程](../tutorials/import.md)。

??? tip "导出你的环境"
    如果你与 Conda 用户或系统合作，你可以[将你的环境导出为 `environment.yml`](../reference/cli/pixi/workspace/export.md) 文件来共享它们。
    ```shell
    pixi workspace export conda-environment
    ```
    你还可以导出 [conda 显式规范](../reference/cli/pixi/workspace/export.md)。

## 故障排除

遇到问题了吗？以下是适应 `conda` 工作流程时一些常见问题的解决方案：

- 依赖项 `由于严格的 channel 优先级被排除，不使用此选项：'https://conda.anaconda.org/conda-forge/'`
  当包在多个 channel 中时会发生此错误。`pixi` 使用严格的 channel 优先级。请参阅 [channel 优先级](../advanced/channel_logic.md) 了解更多。
- `pixi global install pip`，pip 不工作。
  `pip` 安装在全局隔离环境中。在 workspace 中使用 `pixi add pip` 在 workspace 环境中安装 pip 并使用该 workspace。
- `pixi global install <任何库>` -> `import <任何库>` -> `ModuleNotFoundError: 没有名为 '<任何库>' 的模块`
   库安装在全局隔离环境中。在 workspace 中使用 `pixi add <任何库>` 在 workspace 环境中安装库并使用该 workspace。