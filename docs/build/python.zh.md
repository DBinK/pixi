本教程将向您展示如何使用 Pixi 创建一个简单的 Python 包。要了解更多关于如何使用 Pixi 构建包的信息，请参阅[入门指南](./getting_started.md)。您可能还想查看 `pixi-build-python` 后端的[文档](backends/pixi-build-python.md)。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    请在为您的项目使用时牢记这一点。

## 为什么这很有用？

Pixi 基于 conda 生态系统构建，允许您创建包含所有依赖项的 Python 环境。
与 PyPI 不同，conda 生态系统是跨语言的，还提供用 Rust、R、C、C++ 和许多其他语言编写的包。

通过使用 Pixi 构建 Python 包，您可以：

1. 在同一工作区（workspace）中管理 Python 包和其他语言编写的包
2. 使用同一工具构建 conda 和 Python 包

本教程将重点介绍第 1 点。

## 开始

首先，我们创建一个简单的 Python 包，包含一个 `pyproject.toml` 和一个单独的 Python 文件。
该包被称为 `python_rich`，因此我们将创建以下结构：

```shell
├── src # (1)!
│   └── python_rich
│       └── __init__.py
└── pyproject.toml
```

1. 此项目使用 src-layout，但 Pixi 同时支持 [flat- 和 src-layouts](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/#src-layout-vs-flat-layout)。

Python 包有一个单独的函数 `main`。
调用它将打印一个包含三个人姓名、年龄和城市的表格。

```py title="src/python_rich/__init__.py"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/python/src/python_rich/__init__.py"
```

Python 包的元数据在 `pyproject.toml` 中定义。

```toml title="pyproject.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/python/pyproject.toml"
```

1. 我们使用 `rich` 包在终端中打印表格。
2. 通过指定脚本，可执行文件 `rich-example-main` 将在环境中可用。调用它将依次调用 `python_rich` 模块的 `main` 函数。
3. 您可以选择多个后端来构建 Python 包，我们选择 `hatchling`，它无需额外配置即可正常工作。

### 添加 `pixi.toml`

目前我们拥有的已经构成一个完整的 Python 包。
它可以按原样上传到 [PyPI](https://pypi.org/)。

但是，我们仍然需要工具来管理我们的环境，如果我们希望其他 Pixi 项目依赖我们的工具，我们需要包含更多信息。
我们将通过创建 `pixi.toml` 来做到这一点。

!!! note
    Pixi 项目清单（manifest）可以在自己的 `pixi.toml` 文件中，也可以集成在 `pyproject.toml` 中
    在本教程中，我们将使用 `pixi.toml`。
    如果您希望将所有内容集成在 `pyproject.toml` 中，只需将本教程中的 `pixi.toml` 内容复制到您的 `pyproject.toml` 中，并在每个表前加上 `tool.pixi.`。

让我们初始化一个 Pixi 项目。

```
pixi init --format pixi
```

我们传递 `--format pixi` 以便告知 Pixi 我们想要一个 `pixi.toml` 而不是扩展 `pyproject.toml`。

```shell
├── src
│   └── python_rich
│       ├── __init__.py
├── .gitignore
├── pixi.toml
└── pyproject.toml
```

这是 `pixi.toml` 的内容：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/python/pixi.toml"
```

1. 在 `workspace` 中设置跨工作区所有包共享的信息。
2. 在 `dependencies` 中指定您的所有 Pixi 包。这里，这只包括我们自己的包，它在下面的 `package` 中定义
3. 我们定义一个任务来运行我们之前定义的 `rich-example-main` 可执行文件。您可以在此[部分](../workspace/advanced_tasks.md)了解更多关于任务的信息。
4. 在 `package` 中定义实际的 Pixi 包。当其他 Pixi 包或工作区依赖我们的包或当我们将其上传到 conda 通道（channel）时，将使用此信息。
5. 同样，Python 使用构建后端来构建 Python 包，Pixi 使用构建后端来构建 Pixi 包。`pixi-build-python` 从 Python 包创建一个 Pixi 包。
6. 在 `package.host-dependencies` 中，我们添加构建 Python 包所需的 Python 依赖项。通过在此处添加它们，依赖项将来自 conda 通道（channel）而不是 PyPI。
7. 在 `package.run-dependencies` 中，我们添加运行时所需的 Python 依赖项。

现在当我们运行 `pixi run start` 时，我们得到以下输出：

```
┏━━━━━━━━━━━━━━┳━━━━━┳━━━━━━━━━━━━━┓
┃ name         ┃ age ┃ city        ┃
┡━━━━━━━━━━━━━━╇━━━━━╇━━━━━━━━━━━━━┩
│ John Doe     │ 30  │ New York    │
│ Jane Smith   │ 25  │ Los Angeles │
│ Tim de Jager │ 35  │ Utrecht     │
└──────────────┴─────┴─────────────┘
```

## 结论

在本教程中，我们创建了一个基于 Python 的 Pixi 包。
它可以按原样使用，上传到 conda 通道（channel）或 PyPI。
在另一个教程中，我们将学习如何将多个 Pixi 包添加到同一工作区（workspace）并让一个 Pixi 包使用另一个。

感谢阅读！快乐编程 🚀

有任何问题？请随时在 [X](https://twitter.com/prefix_dev) 上联系或分享本教程，[加入我们的 Discord](https://discord.gg/kKV8ZxyzY4)，发送[电子邮件](mailto:hi@prefix.dev)或关注我们的 [GitHub](https://github.com/prefix-dev)。
