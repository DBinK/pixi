如果将包添加到特征的[依赖表](../reference/pixi_manifest.md#dependencies)中，该依赖将在包含该特征的所有环境中可用。
正在构建的包的依赖粒度更细。
在这里您可以看到一个简单 C++ 包的三种依赖类型。

```toml
--8<-- "docs/source_files/pixi_tomls/dependency_types.toml:dependencies"
```

每个依赖在包构建过程的不同步骤中使用。
`cxx-compiler` 用于构建包，`catch` 将链接到包中，`git` 将在运行时可用。

让我们深入研究各种类型的包依赖及其在构建过程中的特定角色。

### [构建依赖（Build Dependencies）](../reference/pixi_manifest.md#build-dependencies)
!!! note "pixi-build-cmake"
    使用 `pixi-build-cmake` 后端时，您不需要将 `cmake` 或编译器指定为依赖项。
    后端默认会安装 `cmake`、`ninja` 和 C++ 编译器。

此表包含构建工作区所需的依赖项。
与依赖项和 host 依赖项不同，这些包是针对构建机器的架构安装的。
这支持从一种机器架构交叉编译到另一种机器架构。

典型的构建依赖示例包括：

- 编译器在构建机器上调用，但它们为目标机器生成代码。
  如果包是交叉编译的，构建和目标机器的架构可能不同。
- `cmake` 在构建机器上调用以生成额外文件，然后将其包含在编译过程中。

!!! info
    _build_ 目标指的是将执行构建的机器。
    这些依赖项安装的程序和库将在构建机器上执行。

    例如，如果您在配备 Apple Silicon 芯片的 MacBook 上编译但针对 Linux x86_64，那么您的 *build* 平台是 `osx-arm64`，您的 *host* 平台是 `linux-64`。

### [Host 依赖（Host Dependencies）](../reference/pixi_manifest.md#host-dependencies)

Host 依赖是在构建/链接期间需要的、特定于宿主机器的依赖。
例如在交叉编译时，与构建依赖的区别就变得重要起来。
编译器是构建依赖项，因为它是您特定于您的机器。
相反，您链接的库是 host 依赖项，因为它们特定于 host 机器。
典型的 host 依赖示例包括：

- 基础解释器：Python 包将在此处列出 `python`，R 包将列出 `mro-base` 或 `r-base`
- 您的包链接的库，如 `openssl`、`rapidjson` 或 `xtensor`

#### Python 代码
由于当前构建的方式，像 `hatchling`、`pip`、`uv` 等依赖项是 host 依赖项。
否则，它会在构建过程中使用错误的 python 前缀。

这是一个技术限制，我们正在寻找方法来减少这种麻烦。
但现在，您需要将这些依赖项添加到 `host-dependencies` 部分。

例如，假设我们想使用 `hatchling` 和 `uv` 来构建 python 包。
您需要在项目清单（manifest）文件中使用类似这样的内容：

```toml
[host-dependencies]
hatchling = "*"
uv = "*"
```

#### 本地代码
在交叉编译时，您可能需要指定应该具有*目标*机器架构的 host 依赖项，并在构建过程中使用它们。
例如，在链接库时。
让我们回顾一下可以在此处找到的说明[A Master Guide To Linux Cross-Compiling](https://ruvi-d.medium.com/a-master-guide-to-linux-cross-compiling-b894bf909386)

- *build machine*：构建代码的地方
- *host machine*：运行构建代码的地方
- *target machine*：构建代码输出的二进制文件运行的地方

假设我们使用 Linux PC (linux-64) 交叉编译一个名为 `Awesome` 的 CMake 应用程序，以在 Linux ARM 目标机器 (linux-aarch64) 上运行。
我们会得到下表：

| Component |    Type     | Build  |  Host  | Target |
|-----------|-------------|--------|--------|--------|
| GCC       | Compiler    | x86_64 | x86_64 | aarch64|
| CMake     | Build tool  | x86_64 | x86_64 | N/A    |
| Awesome   | Application | x86_64 | aarch64  | N/A  |

所以如果我需要使用像 SDL2 这样的库，我需要将其添加到 `host-dependencies` 表中。
因为运行 `Awesome` 的机器的 host 架构与构建架构不同。

在您的项目清单（manifest）文件中给您类似这样的内容：

```toml
  # in our example these dependencies will use the aarch64 binaries
[host-dependencies]
sdl2 = "*"
```

#### 运行导出（Run-Exports）

Conda 包可以定义 `run-exports`，这些依赖项在 `host-dependencies` 部分指定时将隐式添加到 `run-dependencies` 部分。
这很有用，因为可以避免在两个部分中指定相同的依赖项。
因为 conda-forge 上的大多数包都定义了这些 `run-exports`。
使用 `zlib` 这样的东西，您只需要在 `host-dependencies` 部分指定它，它就会自动用作运行依赖项。

### [依赖项（运行依赖项）](../reference/pixi_manifest.md#dependencies)

这些是运行包时需要的依赖项，它们是最常见的依赖项。
也是您在 `workspace` 中通常使用的。
