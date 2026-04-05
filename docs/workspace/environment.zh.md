# 环境

Pixi 是一个管理环境的工具。
本文档解释环境的外观以及如何使用它。

## 激活

环境不过是一组安装在特定位置的文件，某种程度上模仿全局系统安装。
你需要激活环境才能使用它。
在最简单的情况下，这意味着将环境的 `bin` 目录添加到 `PATH` 变量。
但在 conda 环境中还有更多，因为它还设置一些环境变量。

激活有以下几种选择：

- `pixi shell`：启动一个已激活环境的 shell。
- `pixi shell-hook`：打印在当前 shell 中激活环境的命令。
- `pixi run` 在默认环境中运行命令或[任务](./advanced_tasks.md)。

其中 `run` 命令比较特殊，因为它运行自己的跨平台 shell 并能够运行任务。
有关任务的更多信息请参阅[tasks 文档](./advanced_tasks.md)。

在 Pixi 中使用 `pixi shell-hook`，你将获得以下输出：

```shell
export PATH="/home/user/development/pixi/.pixi/envs/default/bin:/home/user/.local/bin:/home/user/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/home/user/.pixi/bin"
export CONDA_PREFIX="/home/user/development/pixi/.pixi/envs/default"
export PIXI_PROJECT_NAME="pixi"
export PIXI_PROJECT_ROOT="/home/user/development/pixi"
export PIXI_PROJECT_VERSION="0.12.0"
export PIXI_PROJECT_MANIFEST="/home/user/development/pixi/pixi.toml"
export CONDA_DEFAULT_ENV="pixi"
export PIXI_ENVIRONMENT_PLATFORMS="osx-64,linux-64,win-64,osx-arm64"
export PIXI_ENVIRONMENT_NAME="default"
export PIXI_PROMPT="(pixi) "
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-binutils_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-gcc_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-gfortran_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/activate-gxx_linux-64.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/libglib_activate.sh"
. "/home/user/development/pixi/.pixi/envs/default/etc/conda/activate.d/rust.sh"
```

它设置 `PATH` 和一些更多的环境变量。但更重要的是，它还运行由已安装包呈现的激活脚本。
一个例子是 [`libglib_activate.sh`](https://github.com/conda-forge/glib-feedstock/blob/52ba1944dffdb2d882d824d6548325155b58819b/recipe/scripts/activate.sh) 脚本。
因此，仅将 `bin` 目录添加到 `PATH` 是不够的。

激活使用的 shell：
- 在 Windows 上，Pixi 在 `cmd.exe` 下执行激活。
- 在 Linux 和 macOS 上，Pixi 在 `bash` 下执行激活。

这同时影响 `[activation.env]` 和 `activation.scripts`：它们在激活期间由平台的 shell 应用，在任何任务运行之前。

你可以通过 manifest 中的 `activation` 表修改激活，你可以添加更多激活脚本或将环境变量注入激活脚本。
```toml
--8<-- "docs/source_files/pixi_tomls/activation.toml:activation"
```
在 [这里](../reference/pixi_manifest.md#the-activation-table) 找到 `activation` 表的参考。

--8<-- "docs/partials/conda-style-activation.md"


## 结构

所有 Pixi 环境默认位于 workspace 的 `.pixi/envs` 目录中。
这保持你的机器和 workspace 干净且相互隔离，并使 workspace 完成后易于清理。
虽然通常推荐这种结构，但也可以通过启用[分离环境](../reference/pixi_configuration.md#detached-environments)将环境存储在 workspace 目录之外。

如果你查看 `.pixi/envs` 目录，你会看到每个环境的目录，如果不指定自定义环境，则使用 `default`，如果你指定了自定义环境，则使用你指定的名称。

```shell
.pixi
└── envs
    ├── cuda
    │   ├── bin
    │   ├── conda-meta
    │   ├── etc
    │   ├── include
    │   ├── lib
    │   ...
    └── default
        ├── bin
        ├── conda-meta
        ├── etc
        ├── include
        ├── lib
        ...
```

这些目录是 conda 环境，你可以按原样使用它们，但不能手动编辑，它们应始终通过 `pixi.toml` 进行。
Pixi 始终确保环境与 `pixi.lock` 文件保持同步。
如果情况并非如此，则使用环境的所有命令都会自动更新它，例如 `pixi run`、`pixi shell`。

### 环境元数据
创建环境时，Pixi 会添加一个包含一些元数据的小文件。
这个文件称为 `pixi`，位于环境的 `conda-meta` 文件夹中。
此文件包含以下信息：

- `manifest_path`：描述用于创建此环境的 workspace 的 manifest 文件路径
- `environment_name`：环境名称
- `pixi_version`：用于创建此环境的 Pixi 版本
- `environment_lock_file_hash`：用于创建此环境的 `pixi.lock` 文件的哈希值

```json
{
  "manifest_path": "/home/user/dev/pixi/pixi.toml",
  "environment_name": "default",
  "pixi_version": "0.34.0",
  "environment_lock_file_hash": "4f36ee620f10329d"
}
```

`environment_lock_file_hash` 用于检查环境是否与 `pixi.lock` 文件同步。
如果 `pixi.lock` 文件的哈希值与 `pixi` 文件中的哈希值不同，Pixi 将更新环境。

这用于加速激活，要触发完全重新验证和安装，请使用 `pixi install` 或 `pixi reinstall`。
损坏的环境通常不会通过哈希比较被发现，但重新验证将重新安装环境。
默认情况下，所有修改锁文件的命令都会触发重新验证，`pixi install` 也会如此。

### 清理

如果要清理环境，你可以简单地删除 `.pixi/envs` 目录，Pixi 将在需要时重新创建环境。

```shell
pixi clean
# or manually:
rm -rf .pixi/envs

# or per environment:
pixi clean --environment cuda
# or manually:
rm -rf .pixi/envs/default
rm -rf .pixi/envs/cuda
```

## 求解环境

当你运行使用环境的命令时，Pixi 会检查它是否与 `pixi.lock` 文件同步。
如果不是，Pixi 将求解环境并更新它。
这意味着 Pixi 将检索你在 `pixi.toml` 中指定的依赖要求的最佳包集，并将求解步骤的输出放入 `pixi.lock` 文件中。
求解是一个数学问题，可能需要一些时间，但我们为我们的求解方式感到自豪，并相信我们可以在合理的时间内解决你的环境。
如果你想了解有关求解过程的更多信息，可以阅读这些：

- [Rattler(conda) resolver blog](https://prefix.dev/blog/the_new_rattler_resolver)
- [UV(PyPI) resolver blog](https://astral.sh/blog/uv-unified-python-packaging)

Pixi 同时求解 `conda` 和 `PyPI` 依赖，其中 `PyPI` 依赖使用 conda 包作为基础，因此你可以确保包彼此兼容。
这些求解器分别在 [`rattler`](https://github.com/conda/rattler) 和 [`uv`](https://github.com/astral-sh/uv) 库中，它们控制求解过程的繁重工作，由我们的自定义 SAT 求解器执行：[`resolvo`](https://github.com/mamba-org/resolvo)。
`resolvo` 能够求解多个生态系统，如 `conda` 和 `PyPI`。它为 `PyPI` 包实现惰性求解过程，这意味着它只下载求解环境所需的包的元数据。
它还支持 `conda` 的求解方式，即一次下载所有包的元数据，然后一次性求解。

对于 `[pypi-dependencies]`，`uv` 实现 `sdist` 构建以检索包的元数据，以及 `wheel` 构建以安装包。
对于此构建步骤，`pixi` 需要首先在 (conda)`[dependencies]` 部分的 `pixi.toml` 文件中安装 `python`。
这总是比纯 conda 求解慢。所以为了获得最佳 Pixi 体验，你应该留在 `pixi.toml` 文件的 `[dependencies]` 部分。

## 缓存包

Pixi 将所有先前下载的包缓存在缓存文件夹中。
此缓存文件夹在所有 Pixi workspace 和全局安装的工具之间共享。

通常位置如下，平台特定的默认缓存文件夹：

- Linux: `$XDG_CACHE_HOME/rattler` 或 `$HOME/.cache/rattler`
- macOS: `$HOME/Library/Caches/rattler`
- Windows: `%LOCALAPPDATA%\rattler`

可以通过设置 `PIXI_CACHE_DIR` 或 `RATTLER_CACHE_DIR` 环境变量来配置此位置。

当你想清理缓存时，你可以简单地删除缓存目录，Pixi 将在需要时重新创建缓存。

缓存包含多个文件夹，涉及 pixi 中不同的缓存。

- `pkgs`：包含下载/解包的 conda 包。
- `repodata`：包含 conda repodata 缓存。
- `uv-cache`：包含 uv 缓存。这包括多个缓存，例如 `built-wheels` `wheels` `archives`
- `http-cache`：包含 conda-pypi 映射缓存。

### 去重

当 Pixi 将包安装到环境时，它不会从缓存复制文件。
相反，它创建**硬链接**（或在支持写时复制的文件系统上创建**reflinks**，例如 macOS 上的 APFS 和 Linux 上的 btrfs/XFS）。
这意味着使用相同版本的包的环境共享相同的磁盘文件，因此包实际上只存储一次。

例如，如果三个 workspace 都依赖 `numpy 1.26.4`，该包的单个文件在缓存中存在一次，并链接到每个环境中。
这可以节省大量磁盘空间，尤其是当你使用许多环境或大型包（如 CUDA 工具包）时。

!!! note "硬链接 vs reflinks"
    - **硬链接** 指向磁盘上的同一数据。修改一个链接会修改所有链接，但这不是问题，因为 Pixi 环境是只读的。
    - **Reflinks**（写时复制链接）表现得像即时复制，只在一方被修改时才分配新的磁盘空间。这使它们比硬链接更安全：如果进程意外写入环境中的文件，只有该环境的副本受影响，而缓存的原始文件和所有其他环境保持不变。在支持的文件系统上，Pixi 因此更喜欢 reflinks。
    - 如果硬链接或 reflinks 不可用（例如，当缓存和 workspace 在不同的挂载点上），Pixi 会回退到复制文件。