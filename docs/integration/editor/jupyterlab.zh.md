## 基本使用

使用 Pixi 使用 JupyterLab 非常简单。
你只需创建一个新的 Pixi 工作区并将 `jupyterlab` 包添加到其中。
完整示例在以下 [Github 链接](https://github.com/prefix-dev/pixi/tree/main/examples/jupyterlab)中提供。

```bash
pixi init
pixi add jupyterlab
```

这将创建一个新的 Pixi 工作区并将 `jupyterlab` 包添加到其中。然后你可以使用以下命令启动 JupyterLab：

```bash
pixi run jupyter lab
```

如果你想向 JupyterLab 添加更多"内核"，你可以简单地将它们添加到当前工作区，以及你可能需要的科学栈的任何依赖。

```bash
pixi add bash_kernel ipywidgets matplotlib numpy pandas  # ...
```

### 有哪些内核可用？

你可以轻松地为 JupyterLab 安装更多"内核"。`conda-forge` 仓库有許多有趣的内核 - 不仅仅是 Python！

- [**`bash_kernel`**](https://prefix.dev/channels/conda-forge/packages/bash_kernel) Bash 内核
- [**`xeus-cpp`**](https://prefix.dev/channels/conda-forge/packages/xeus-cpp) 基于新的 clang-repl 的 C++ 内核
- [**`xeus-cling`**](https://prefix.dev/channels/conda-forge/packages/xeus-cling) 基于稍旧的 Cling 的 C++ 内核
- [**`xeus-lua`**](https://prefix.dev/channels/conda-forge/packages/xeus-lua) Lua 内核
- [**`xeus-sql`**](https://prefix.dev/channels/conda-forge/packages/xeus-sql) SQL 内核
- [**`r-irkernel`**](https://prefix.dev/channels/conda-forge/packages/r-irkernel) R 内核

## 高级使用

<!--
Modifications to the following section are related to the README.md in https://github.com/renan-r-santos/pixi-kernel and
https://github.com/renan-r-santos/pixi-kernel-binder, please keep these two in sync by making a PR in both
-->

如果你只想运行一个 JupyterLab 实例，但仍然想要每个目录的 Pixi 环境，你可以使用 [**`pixi-kernel`**](https://prefix.dev/channels/conda-forge/packages/pixi-kernel) 包提供的内核之一。

### 配置 JupyterLab

首先，创建一个 Pixi 工作区，添加 `jupyterlab` 和 `pixi-kernel`，然后启动 JupyterLab：

```bash
pixi init
pixi add jupyterlab pixi-kernel
pixi run jupyter lab
```

这将启动 JupyterLab 并在浏览器中打开它。

![JupyterLab launcher screen showing Pixi
Kernel](https://raw.githubusercontent.com/renan-r-santos/pixi-kernel/main/assets/launch-light.png#only-light)
![JupyterLab launcher screen showing Pixi
Kernel](https://raw.githubusercontent.com/renan-r-santos/pixi-kernel/main/assets/launch-dark.png#only-dark)

`pixi-kernel` 在笔记本的同一目录或任何父目录中查找 manifest 文件（`pixi.toml` 或 `pyproject.toml`）。找到后，它将使用 manifest 文件中指定的环境来启动内核并运行你的笔记本。
