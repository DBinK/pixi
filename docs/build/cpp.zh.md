此示例展示如何使用 CMake 和 `pixi-build` 构建 C++ 包。要了解更多关于如何使用 Pixi 构建包的信息，请参阅[入门指南](./getting_started.md)。您可能还想查看 `pixi-build-cmake` 后端的[文档](backends/pixi-build-cmake.md)。

我们将首先创建一个使用 [nanobind](https://github.com/wjakob/nanobind) 构建 Python 绑定的工作区（workspace）。
我们也可以使用 Pixi 对其进行测试。稍后我们将此示例与 Python 包结合使用。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    请在为您的工作区（workspace）使用它时牢记这一点。

## 创建新工作区（workspace）

首先，使用 Pixi 创建一个新工作区（workspace）：

```bash
pixi init cpp_math
```

这应该会给您一个基本的 `pixi.toml` 以开始使用。

现在我们将创建以下源目录结构：
```bash
cpp_math/
├── CMakeLists.txt
├── pixi.toml
├── .gitignore
└── src
    └── math.cpp
```

## 创建工作区文件
接下来我们将创建：

- `pixi.toml` 文件，用于配置 Pixi
- `CMakeLists.txt` 文件，用于构建绑定
- `src/math.cpp` 文件，包含绑定代码

### `pixi.toml` 文件
使用以下 `pixi.toml` 文件，您可以将鼠标悬停在注释上查看添加每个步骤的原因。

```toml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/cpp/pixi.toml"
```

1. 添加启用 Pixi 构建包的 **preview** 功能 `pixi-build`
2. 这些是工作区（workspace）依赖项。我们添加我们自己的包以及 Python，以便稍后可以运行我们的包
3. 让我们添加一个将运行我们的测试的任务
4. 这是我们指定包名称和版本的地方。
   此部分表示此 `pixi.toml` 文件同时定义了工作区（workspace）和包
5. 我们使用 `pixi-build-cmake` 作为构建系统，以便获得构建 cmake 包的后端
6. 我们使用 [nanobind](https://github.com/wjakob/nanobind) 包来构建我们的绑定
7. 我们需要 python 来构建绑定，因此我们在 `python` 包上添加一个 host 依赖项
8. 我们覆盖 cmake 版本以确保它与我们的 `CMakeLists.txt` 文件匹配
9. （可选）我们可以向 CMake 调用添加额外参数（例如 `-DCMAKE_BUILD_TYPE=Release` 或 `-DUSE_FOOBAR=True`）。这完全取决于特定的工作区（workspace）/CMakeLists.txt 文件

### `CMakeLists.txt` 文件

接下来让我们添加 `CMakeList.txt` 文件：
```CMake
--8<-- "docs/source_files/pixi_workspaces/pixi_build/cpp/CMakeLists.txt"
```

1. 查找 `python`，这实际上找到任何高于 3.8 的版本，但我们使用 3.8 作为最低版本
2. 因为我们在 conda 环境中使用 `python`，我们需要查询 python 解释器来查找 `nanobind` 包
3. 因为我们希望安装目录独立于 python 版本，我们查询 python 的 `site-packages` 目录
4. 查找已安装的 nanobind 包
5. 使用我们的绑定文件作为源文件
6. 将绑定安装到指定的环境前缀（prefix）

### `src/math.cpp` 文件

接下来让我们添加 `src/math.cpp` 文件，这个相当简单：

```cpp
--8<-- "docs/source_files/pixi_workspaces/pixi_build/cpp/src/math.cpp"
```

1. 我们定义一个用于将两个数字相加的函数
2. 我们使用 `nanobind` 包将此函数绑定到 python 模块

## 测试一切是否正常
现在我们已经创建了文件，我们可以测试一切是否正常：

```bash
$ pixi run start
3
```

此命令构建绑定，安装它们，然后运行 `test` 任务。

## 结论

在本教程中，我们创建了一个使用 C++ 的 Pixi 包。
它可以按原样使用，上传到 conda 通道（channel）。
在另一个教程中，我们将学习如何将多个 Pixi 包添加到同一工作区（workspace）并让一个 Pixi 包使用另一个。

感谢阅读！快乐编程 🚀

有任何问题？请随时在 [X](https://twitter.com/prefix_dev) 上联系或分享本教程，[加入我们的 Discord](https://discord.gg/kKV8ZxyzY4)，[发送电子邮件](mailto:hi@prefix.dev)给我们或关注我们的 [GitHub](https://github.com/prefix-dev)。
