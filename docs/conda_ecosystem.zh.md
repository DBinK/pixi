# Conda 生态系统

现在你已经[创建了 workspace](first_workspace.md) 并看到了
[Pixi 的基本命令](getting_started.md)，你可能已经注意到了
channel、subdir 和 platform 等术语。Pixi 构建在
conda 包管理生态系统之上，本页解释这些术语的含义以及它们如何协同工作。

## 是什么让 conda 与众不同？

语言包管理器（pip、npm、cargo）只处理它们自己的语言。
系统包管理器（apt、dnf、pacman）覆盖一切，但安装到一个单一的全局包集合——你无法让同一个库的两个版本并存。
Nix 解决了版本控制问题，但学习曲线陡峭，且有自己独特的配置语言。

Conda 介于两者之间：它像系统包管理器一样**与语言无关**——单个依赖树可以混合 C 库、编译器、CLI 工具和语言运行时——但它像语言包管理器一样安装到**隔离的环境**中。

还有其他几点使其与众不同：

- **仅二进制分发。** 包始终是预编译的，因此安装快速且不需要在用户机器上安装构建工具链。
- **操作系统独立。** 包可以支持常见的操作系统，如 Linux、macOS、Windows 等。
- **对主机硬件的良好支持。** 包可以非常精确地定制以处理可用的硬件。
- **独立的环境。** 每组请求的包都安装到一个自包含的目录中。不与系统包或其他环境共享任何东西。
- **所有版本同时可用。** channel 保留每个发布的版本和构建。求解器选择正确的组合——你可以在一个项目中固定 `numpy 1.24`，在另一个项目中使用 `numpy 2.1`，而不会冲突，甚至可以在一个项目中的不同环境中有不同版本的 `numpy`。

## 包

**conda 包**是一个存档
（[`.conda`](https://conda.io/projects/conda/en/latest/user-guide/concepts/packages.html)
或遗留 `.tar.bz2`），包含预编译的文件和元数据。
元数据声明包的名称、版本、依赖项以及它是为哪个平台构建的。
求解器读取此元数据来决定安装什么——在解析过程中包本身从不执行。

### Subdir：conda 如何定位平台

conda 包针对特定的**平台**（操作系统 + 架构组合）进行编译。
在 conda 术语中，每个平台称为一个 **subdir**——
是 subdirectory 的缩写，因为 channel 将包存储在以它们针对的平台命名的目录中：

```
conda-forge/
├── linux-64/        # Linux on x86_64
├── linux-aarch64/   # Linux on ARM64 (e.g. Graviton, Raspberry Pi 5)
├── osx-64/          # macOS on Intel
├── osx-arm64/       # macOS on Apple Silicon
├── win-64/          # Windows on x86_64
├── win-arm64/       # Windows on ARM64
└── noarch/          # platform-independent packages
```

当求解器解析依赖时，它会查看与目标平台匹配的 subdir 中的包，以及 `noarch`。
`noarch/` 中的包可以在每个平台上工作——这些通常是纯 Python 包或只包含数据的包。

在 Pixi workspace 中，你声明支持哪些平台：

```toml title="pixi.toml"
[workspace]
platforms = ["linux-64", "osx-arm64", "win-64"]
```

Pixi 为每个列出的平台解析依赖，并将结果记录在锁文件中，
即使是你当前不在其上运行的平台。这就是跨平台锁文件成为可能的原因。
请参阅[多平台配置](workspace/multi_platform_configuration.md)了解平台特定的依赖和激活脚本。

### Variant：同一包的多个构建

一个包的单个版本可以以多种方式构建。
例如，`numpy 2.1.0` 可能分别为 Python 3.11 和 Python 3.12 提供单独的构建，
或者 `pytorch 2.4.0` 可能同时有 CPU 和 CUDA 构建。
在 conda 生态系统中，同一版本的不同构建称为 **variant**。

每个 variant 都有一个唯一的**构建字符串**，编码使它不同的内容——通常是 Python 版本、构建配置的哈希值和构建编号：

```
numpy-2.1.0-py311h43a39b2_0.conda
            ^^^^^ ^^^^^^^^ ^
            │     │        └─ build number
            │     └────────── configuration hash
            └──────────────── Python version
```

求解器会根据你的环境中已有的内容自动选择正确的 variant。
如果你已解析 Python 3.12，它会选择 numpy 的 `py312` variant。
你也可以使用 [MatchSpec](concepts/package_specifications.md) 语法固定特定的构建：

```shell
pixi add "pytorch [build='cuda*']"
```

当使用 pixi-build **构建**包时，你可以定义 variant 以从单个源产生多个构建。
有关教程，请参阅[构建变体](build/variants.md)。

### 虚拟包：描述主机系统

某些包需要不是由其他包提供而是由**主机系统**本身提供的功能——最低 Linux 内核版本、特定的 glibc 或 NVIDIA GPU 驱动程序。
conda 将这些系统能力建模为**虚拟包**：特殊包的名字以 `__` 开头，求解器像对待任何其他依赖一样对待它们，但永远不会下载或安装。

常见的虚拟包：

| 虚拟包    | 代表的含义                |
|----------|-------------------------|
| `__linux`       | Linux 内核版本 |
| `__glibc`       | GNU C 库版本 |
| `__osx`         | macOS 版本 |
| `__cuda`        | NVIDIA 驱动程序的 CUDA 能力 |
| `__archspec`    | CPU 微架构（例如 `x86_64_v3`）|

当一个包依赖 `__cuda >= 12` 时，求解器只会选择系统提供 `__cuda` 虚拟包且版本为 12 或更高的环境。这是生态系统防止在不兼容 GPU 驱动程序的机器上安装 GPU 加速包的方式。

Pixi 会自动检测你系统的虚拟包。
你也可以在 workspace 中显式声明它们——例如，告诉求解器你的部署目标有 CUDA 12：

```toml title="pixi.toml"
[system-requirements]
cuda = "12"
```

请参阅[系统要求](workspace/system_requirements.md)了解默认值、每 Feature 覆盖和环境变量覆盖。

## 环境与前缀

你已经在[第一个 Workspace](first_workspace.md)中看到 Pixi 为你的 workspace 创建了一个环境。
在 conda 术语中，环境也称为**前缀**——作为环境根目录的目录路径（安装前缀）。
conda 和 mamba 之类的工具通常将前缀存储在 `~/miniconda3/envs/` 下。
Pixi 将它们保留在 workspace 目录内（`.pixi/envs/`），因此每个项目都有自己的隔离环境，易于查找和清理。

一个 workspace 可以定义[多个环境](workspace/multi_environment.md)——例如，
一个用于开发的 `default` 环境，一个包含额外测试依赖的 `test` 环境，以及一个用于构建文档的 `docs` 环境。
请参阅[环境](workspace/environment.md)了解更多详情。

Pixi 还可以[全局](global_tools/introduction.md)安装工具——在任意 workspace 之外。
全局工具如 `git`、`ripgrep` 或 `ipython` 各自获得自己的隔离前缀，
因此它们不会相互冲突，也不会与你的项目环境冲突。

## Channel：包的存放位置

**channel** 是通过 HTTP 服务的 conda 包仓库。
当你添加依赖时，Pixi 会从一个或多个 channel 获取它。最重要的有：

- **[conda-forge](https://conda-forge.org/)** - 社区维护的默认 channel，包含超过 30,000 个包。这是大多数开源软件的所在地。
- **[bioconda](https://bioconda.github.io/)** - 专注于生物医学研究的社区维护 channel。
- **[robostack-<ros-distro>](https://robostack.github.io/)** - 用于在 conda 生态系统中使用 ROS 包的社区维护 channel。
- **[prefix.dev/github-releases](https://prefix.dev/channels/github-releases)** - 自动从 GitHub releases 生成的 conda 包，使你可以安装还没有 conda-forge 配方的 CLI 工具。

所有这些 channel 都可以在 **[prefix.dev](https://prefix.dev/)** 上发现和浏览，或[托管你自己的私有 channel](https://prefix.dev/channels/)。

你可以在一个 workspace 中混合 channel。
例如，[PyTorch 教程](python/pytorch.md) 展示如何组合不同的 channel。
请参阅[频道逻辑](advanced/channel_logic.md)了解 channel 优先级的工作细节。

## 生态系统中的工具

有几个工具可以管理 conda 包：

| 工具 | 说明 |
|------|-------------|
| **[conda](https://docs.conda.io/)** | 最初的包管理器，用 Python 编写。是 Anaconda 发行版的一部分。 |
| **[mamba](https://mamba.readthedocs.io/)** | 更快的 conda 替代品，使用 C++ 求解器。 |
| **Pixi** | 现代的、基于 Rust 的包管理器，具有锁文件、任务和多环境支持。不是直接替代品——它重新思考了工作流程。 |

如果你来自 conda 或 mamba，请参阅
[从 Conda 切换](switching_from/conda.md) 进行命令对比。

## Rattler 和 rattler-build

Pixi 由 **[Rattler](https://github.com/conda/rattler)** 提供支持，
这是一组用 Rust 从头实现 conda 生态系统的库：
依赖解析、包安装、channel 交互和环境管理。
Rattler 不是 CLI 工具——它是 Pixi（和其他工具）构建于其上的引擎。

在 Rattler **消费**包的地方，有两个工具帮助你**创建**它们：

- **[pixi-build](build/getting_started.md)** 是打包代码最简单的方式。
  它读取你的项目元数据（例如 `pyproject.toml`）并以最少的配置产生 conda 包——适合大多数项目。
- **[rattler-build](https://rattler.build)** 让你对打包过程的每个方面进行细粒度控制：
  自定义构建脚本、修补源文件、交叉编译等。当你需要完全控制包的构建方式时使用它。

两者都作为[构建后端](build/backends/pixi-build-rattler-build.md)与 Pixi 集成，
因此你可以在 Pixi workspace 中构建、测试和发布包。

要更深入地比较 conda 和 PyPI 生态系统以及 Pixi 如何桥接它们，
请参阅 [Conda & PyPI](concepts/conda_pypi.md)。