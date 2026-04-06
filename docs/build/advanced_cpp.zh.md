本教程将向您展示如何使用 [`rattler-build`](https://rattler.build) 构建与[构建 C++ 包](cpp.md)教程中相同的 C++ 包。
本教程假设您已经阅读了[构建 C++ 包](cpp.md)教程。
如果您还没有阅读，建议您先阅读后再继续。
您可能还想查看 `pixi-build-rattler-build` 后端的[文档](backends/pixi-build-rattler-build.md)。
项目结构和源代码与上一个教程相同，因此我们可以跳过对某些部分的详细解释。

当您的语言或构建系统不存在构建后端时，这可能很有用。
另一个原因是当您想对构建过程有更多控制时。

为了说明这一点，我们将使用与上一个教程相同的 C++ 包，但这次我们将使用 `rattler-build` 来构建它。
这将揭示构建过程中隐藏的复杂性，并帮助您更好地理解后端的工作原理。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    请在为您的工作区（workspace）使用它时牢记这一点。

!!! hint
    如果存在后端，请优先使用它。这将为您提供更精简和统一的构建体验。

## 工作区（workspace）结构

首先，请重现上一个教程[构建 C++ 包](cpp.md)中的工作区（workspace）结构。

### `pixi.toml` 文件

我们现在使用 `pixi-build-rattler-build` 后端而不是 `pixi-build-cmake` 后端。

```toml hl_lines="20-21"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/advanced_cpp/pixi.toml"
```

## `recipe.yaml` 文件

接下来让我们添加描述 `rattler-build` 如何构建包的 `recipe.yaml` 文件。
您可以在 `rattler-build` 文档[网页](https://rattler.build/latest/reference/recipe_file/)上找到参考。

```yaml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/advanced_cpp/recipe.yaml"
```

1. 因为我们指定当前目录作为源目录，`rattler-build` 可能会跳过未被 git 跟踪的文件。如果您的文件已经被 git 跟踪，您可以删除此配置。
2. 此构建脚本使用 `Ninja` 构建系统配置和构建 `CMake` 项目。它设置各种选项，如构建类型为 `Release`、安装前缀为 `$PREFIX`、启用共享库和导出编译命令。然后在指定构建目录 `($SRC_DIR/../build)` 中构建项目，并将构建的文件安装到安装目录。
3. 对于构建依赖项，我们需要编译器以及构建系统 `cmake` 和 `ninja`。确保 `cmake` 版本与 `CMakeLists.txt` 文件中的版本匹配。
4. 对于 `python` 绑定，我们需要 `nanobind` 和 `python` 本身。它们被设置在 `host` 依赖部分，因为我们链接到安装绑定的 `python`，而不是构建它。

## 测试一切是否正常

现在我们已经定义了一个 `pixi` 任务，允许我们检查我们的包是否可以正确地加 `1` 和 `2`：

```toml
[tasks]
start = "python -c 'import cpp_math as b; print(b.add(1, 2))'"
```

执行任务按预期工作

```bash
$ pixi run start
3
```

此命令构建绑定，安装它们，然后运行 `test` 任务。

## 结论

在本教程中，我们使用 `rattler-build` 和 `recipe.yaml` 文件创建了一个 Pixi 包。
使用这种方法，我们对构建过程有更多控制。
例如，我们可以使用 `CMAKE_BUILD_TYPE` 将构建类型更改为 `Debug`，通过移除 `-GNinja` 配置使用 `Make` 而不是 `Ninja`。
或者我们可以使用 `make -j$(nproc)` 来指定构建包时并行运行的作业数。

同时，我们失去了语言构建后端所做的大量繁重工作的好处。

感谢阅读！快乐编程 🚀

有任何问题？请随时在 [X](https://twitter.com/prefix_dev) 上联系或分享本教程，[加入我们的 Discord](https://discord.gg/kKV8ZxyzY4)，[发送电子邮件](mailto:hi@prefix.dev)给我们或关注我们的 [GitHub](https://github.com/prefix-dev)。
