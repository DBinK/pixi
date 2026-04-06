本教程将向您展示如何将多个 Pixi 包集成到单个工作区（workspace）中。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    请在为您的项目使用它时牢记这一点。

## 为什么这很有用？

来自 conda 通道（channel）的包已经是构建好的，可以直接使用。
如果您想依赖一个包，因此通常会从这样的通道（channel）获取该包。
但是，在某些情况下您想依赖包的源代码。
例如，如果您想在同一个仓库中开发多个包。
或者如果您需要某个依赖项的未发布版本的更改。

## 开始

在本教程中，我们将展示如何在一个工作区（workspace）中开发两个包。
为此，我们将使用在[构建 Python 包](python.md)章节中开发的 `python_rich` Python 包，并让它依赖在[构建 C++ 包](cpp.md)章节中开发的 `cpp_math` C++ 包。

我们将从 `python_rich` 的原始设置开始，并将 `cpp_math` 复制到一个名为 `packages` 的文件夹中。
源目录结构现在看起来像这样：

```shell
.
├── packages
│   └── cpp_math
│       ├── CMakeLists.txt
│       ├── pixi.toml
│       └── src
│           └── math.cpp
├── pixi.lock
├── pixi.toml
├── pyproject.toml
└── src
    └── python_rich
        └── __init__.py
```

在 Pixi 项目清单（manifest）中，您可以管理工作区（workspace）和/或描述一个包。
在 `python_rich` 的情况下，我们选择同时做这两件事，所以我们只需要将 `cpp_math` 添加为 `python_rich` 的[运行依赖](../reference/pixi_manifest.md#run-dependencies)。

```py title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/workspace/pixi.toml:run-dependencies"
```

我们只想使用顶级项目清单（manifest）的 `workspace` 表。
因此，我们可以删除 `cpp_math` 项目清单（manifest）中的 workspace 部分。

```diff title="packages/cpp_math/pixi.toml"
-[workspace]
-channels = ["https://prefix.dev/conda-forge"]
-platforms = ["osx-arm64", "osx-64", "linux-64", "win-64"]
-preview = ["pixi-build"]
-
-[dependencies]
-cpp_math = { path = "." }
-
-[tasks]
-start = "python -c 'import cpp_math as b; print(b.add(1, 2))'"
```

实际上 `python_rich` 有一个问题。
每个人的年龄都差一岁！

```
┏━━━━━━━━━━━━━━┳━━━━━┳━━━━━━━━━━━━━┓
┃ name         ┃ age ┃ city        ┃
┡━━━━━━━━━━━━━━╇━━━━━╇━━━━━━━━━━━━━┩
│ John Doe     │ 30  │ New York    │
│ Jane Smith   │ 25  │ Los Angeles │
│ Tim de Jager │ 35  │ Utrecht     │
└──────────────┴─────┴─────────────┘
```

我们需要给每个人的年龄加一岁。
幸运的是，`cpp_math` 公开了一个函数 `add`，允许我们做到这一点。

```py title="src/python_rich/__init__.py"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/workspace/src/python_rich/__init__.py"
```

如果您运行 `pixi run start`，现在每个人的年龄应该是准确的：

```
┏━━━━━━━━━━━━━━┳━━━━━┳━━━━━━━━━━━━━┓
┃ name         ┃ age ┃ city        ┃
┡━━━━━━━━━━━━━━╇━━━━━╇━━━━━━━━━━━━━┩
│ John Doe     │ 31  │ New York    │
│ Jane Smith   │ 26  │ Los Angeles │
│ Tim de Jager │ 36  │ Utrecht     │
└──────────────┴─────┴─────────────┘
```

## 结论

在本教程中，我们创建了一个包含两个包的工作区（workspace）。
`python_rich` 的项目清单（manifest）描述了工作区（workspace）和包，对于 `cpp_math` 只使用了 `package` 部分。
随意添加更多用不同语言编写的包到这个工作区（workspace）中！

感谢阅读！快乐编程 🚀

有任何问题？请随时在 [X](https://twitter.com/prefix_dev) 上联系或分享本教程，[加入我们的 Discord](https://discord.gg/kKV8ZxyzY4)，发送[电子邮件](mailto:hi@prefix.dev)或关注我们的 [GitHub](https://github.com/prefix-dev)。
