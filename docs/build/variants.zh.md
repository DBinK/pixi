本教程将向您展示如何使用变体（variants）针对不同版本的依赖项构建 Pixi 包。
有人可能会将此功能称为构建矩阵、构建配置或参数化构建，在 conda 生态系统中这被称为变体（variant）。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    请在为您的项目使用它时牢记这一点。

## 为什么这很有用？

当我们依赖一个 Pixi 包时，该包本身的依赖版本已经设置好了。
例如，在 [C++ 教程](cpp.md) 中，我们构建的 `cpp_math` 包依赖于 Python 3.12。
如果我们将 Python 3.11 和 `cpp_math` 都添加到我们的工作区（workspace），Pixi 会报告版本冲突。
通过使用变体（variants），我们可以为特定依赖项添加一组允许的版本。
然后 Pixi 将使用所有不同的变体来解析包。

## 开始

在本教程中，我们将继续使用[工作区教程](workspace.md)的结果，以便我们可以针对多个 Python 版本进行测试。
提醒一下，我们最终得到了一个包含工作区（workspace）和 Python 包 `python_rich` 的顶级 `pixi.toml`。
然后我们的工作区（workspace）依赖 `python_rich` 和 `cpp_math`。

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/workspace_variants/pixi.toml:dependencies"
```

文件树看起来像这样：

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

为了允许多个 Python 版本，我们首先需要将 `cpp_math` 的 Python 版本要求从 `3.12.*` 更改为 `*`。

```toml title="packages/cpp_math/pixi.toml" hl_lines="4"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/workspace_variants/packages/cpp_math/pixi.toml:host-dependencies"
```

1. 原来是 "3.12.*"

现在，我们需要指定我们想要允许的 Python 版本。
我们在 `workspace.build-variants` 中做到这一点：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/workspace_variants/pixi.toml:variants"
```

如果我们现在运行 `pixi install`，我们将让 Pixi 决定使用 Python 3.11 还是 3.12。
在实践中，您会想要创建多个环境并指定不同的依赖版本。
在我们的情况下，这允许我们针对 Python 3.11 和 3.12 测试我们的设置。

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/workspace_variants/pixi.toml:environments"
```

通过运行 `pixi list`，我们可以看到每个环境中使用的 Python 版本。
您还可以看到 `cpp_math` 的 `Build` 字符串在 `py311` 和 `py312` 之间有所不同。
这意味着已为每个变体构建了不同的包。
由于 `python_rich` 只包含 Python 源代码，单个构建可以用于多个 Python 版本。
该包是 `noarch`（即跨平台）。
因此，构建字符串是相同的。

```pwsh
$ pixi list --environment py311
Package            Version     Build               Size       Kind   Source
python             3.11.11     h9e4cc4f_1_cpython  29.2 MiB   conda  python
cpp_math           0.1.0       py311h43a39b2_0                conda  cpp_math
python_rich        0.1.0       pyhbf21a9e_0                   conda  python_rich
```

```pwsh
$ pixi list --environment py312
Package            Version     Build               Size       Kind   Source
python             3.12.8      h9e4cc4f_1_cpython  30.1 MiB   conda  python
cpp_math           0.1.0       py312h2078e5b_0                conda  cpp_math
python_rich        0.1.0       pyhbf21a9e_0                   conda  python_rich
```

## 结论

在本教程中，我们展示了如何使用变体（variants）构建单个包的多个版本。
我们为 Python 3.12 和 3.13 构建了 `cpp_math`，这允许我们测试它是否在两个 Python 版本上都能正常工作。
变体（variants）不仅限于单个依赖项，例如您可以尝试测试多个版本的 `nanobind`。

除了内联添加变体（variants）外，它们也可以作为文件包含。请查看[参考文档](../reference/pixi_manifest.md#build-variants-files-optional)以了解更多！

感谢阅读！快乐编程 🚀

有任何问题？请随时在 [X](https://twitter.com/prefix_dev) 上联系或分享本教程，[加入我们的 Discord](https://discord.gg/kKV8ZxyzY4)，发送[电子邮件](mailto:hi@prefix.dev)或关注我们的 [GitHub](https://github.com/prefix-dev)。
