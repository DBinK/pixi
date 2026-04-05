# 全局清单

此全局清单包含全局安装的环境列表、它们的依赖项和暴露的二进制文件。
它可以被编辑、同步、检入版本控制系统，并与他人共享。

运行上一部分中的命令会导致以下清单：
```toml
version = 1

[envs.rattler-build]
channels = ["conda-forge"]
dependencies = { rattler-build = "*" }
exposed = { rattler-build = "rattler-build" }

[envs.ipython]
channels = ["conda-forge"]
dependencies = { ipython = "*", numpy = "*", matplotlib = "*" }
exposed = { ipython = "ipython", ipython3 = "ipython3" }

[envs.python]
channels = ["conda-forge"]
dependencies = { python = "3.12.*" } # (1)!
exposed = { py3 = "python" } # (2)!
```

1. 依赖项是要安装到环境中的包。你可以指定版本或使用通配符。
2. 暴露的二进制文件是将在系统路径中可用的文件。在这种情况下，`python` 以名称 `py3` 暴露。

## 清单位置

根据你的操作系统，清单可以在以下位置找到。
运行 [`pixi info`](../reference/cli/pixi/info.md)，找到你系统上当前使用的清单。

=== "Linux"

    | **优先级** | **位置**                                             | **注释**                                  |
    |--------------|----------------------------------------------------------|-----------------------------------------------|
    | 4            | `$PIXI_HOME/manifests/pixi-global.toml`                  | `PIXI_HOME` 中的全局清单。               |
    | 3            | `$HOME/.pixi/manifests/pixi-global.toml`                 | 用户主目录中的全局清单。       |
    | 2            | `$XDG_CONFIG_HOME/pixi/manifests/pixi-global.toml`       | XDG 兼容配置目录。               |
    | 1            | `$HOME/.config/pixi/manifests/pixi-global.toml`          | 配置目录。                             |

=== "macOS"

    | **优先级** | **位置**                                             | **注释**                                  |
    |--------------|----------------------------------------------------------|-----------------------------------------------|
    | 3            | `$PIXI_HOME/manifests/pixi-global.toml`                  | `PIXI_HOME` 中的全局清单。               |
    | 2            | `$HOME/.pixi/manifests/pixi-global.toml`                 | 用户主目录中的全局清单。       |
    | 1            | `$HOME/Library/Application Support/pixi/manifests/pixi-global.toml`| 配置目录。                             |


=== "Windows"

    | **优先级** | **位置**                                             | **注释**                                  |
    |--------------|----------------------------------------------------------|-----------------------------------------------|
    | 3            | `$PIXI_HOME\manifests\pixi-global.toml`                  | `PIXI_HOME` 中的全局清单。               |
    | 2            | `%USERPROFILE%\.pixi\manifests\pixi-global.toml`         | 用户主目录中的全局清单。       |
    | 1            | `%APPDATA%\pixi\manifests\pixi-global.toml`                        | 配置目录。                             |


!!! note
    如果存在多个位置，将使用优先级最高的清单。

## Channel

`channels` 键描述将用于下载包的 Conda channel。
这些有优先级，所以第一个优先级最高。
如果包在该 channel 中未找到，则使用下一个。
例如，运行：
```
pixi global install --channel conda-forge --channel bioconda snakemake
```
结果在清单中产生以下条目：
```toml
[envs.snakemake]
channels = ["conda-forge", "bioconda"]
dependencies = { snakemake = "*" }
exposed = { snakemake = "snakemake" }
```

有关 channel 的更多信息，请参阅[这里](../advanced/channel_logic.md)。

## 依赖

依赖是要安装到环境中的 Conda 包。例如，运行：
```
pixi global install "python<3.12"
```
在清单中创建以下条目：
```toml
[envs.vim]
channels = ["conda-forge"]
dependencies = { python = "<3.12" }
# ...
```
通常，你只指定你正在安装的工具，但可以根据需要添加更多包。
定义要安装到的环境将允许你一次添加多个依赖。例如，运行：
```shell
pixi global install --environment my-env git vim python
```
将在清单中创建以下条目：

```toml
[envs.my-env]
channels = ["conda-forge"]
dependencies = { git = "*", vim = "*", python = "*" }
# ...
```

你可以运行以下命令将依赖[`添加`](../reference/cli/pixi/global/add.md)到现有环境：
```shell
pixi global add --environment my-env package-a package-b
```

它们将作为依赖添加到 `my-env` 环境，但不会自动暴露新包的二进制文件。

你可以运行以下命令[`移除`](../reference/cli/pixi/global/remove.md)依赖：

```shell
pixi global remove --environment my-env package-a package-b
```

## 暴露的可执行文件

你可以指示 `pixi global install` 以什么名称暴露可执行文件：

```shell
pixi global install --expose bird=bat bat
```

清单修改如下：

```toml
[envs.bat]
channels = ["https://prefix.dev/conda-forge"]
dependencies = { bat = "*" }
exposed = { bird = "bat" }
```

这意味着可执行文件 `bat` 将以名称 `bird` 暴露。

### 自动暴露的可执行文件

如果安装与环境同名的包，它将以相同名称暴露，这是一些额外的自动行为。
即使二进制文件仅通过包的依赖暴露也是如此。
例如，运行：
```
pixi global install ansible
```
将在清单中创建以下条目：
```toml
[envs.ansible]
channels = ["conda-forge"]
dependencies = { ansible = "*" }
exposed = { ansible = "ansible" } # (1)!
```

1. `ansible` 二进制文件即使由 `ansible` 的依赖 `ansible-core` 包安装也会被暴露。

也可以暴露位于嵌套目录中的可执行文件。
例如 dotnet.exe 可执行文件位于 dotnet 文件夹中，
要暴露 `dotnet`，你必须指定其相对路径：

```
pixi global install dotnet --expose dotnet=dotnet\dotnet
```

这将在清单中创建以下条目：
```toml
[envs.dotnet]
channels = ["conda-forge"]
dependencies = { dotnet = "*" }
exposed = { dotnet = 'dotnet\dotnet' }
```

## 快捷方式

对于图形用户界面，添加快捷方式特别有用。
这样应用程序会出现在开始菜单中，或者在你想要打开应用程序支持的文件类型时被建议。
如果包支持快捷方式，你这边什么都不需要做。
只需执行 `pixi global install` 即可完成。
例如，`pixi global install mss` 将导致以下清单：

```toml
[envs.mss]
channels = ["https://prefix.dev/conda-forge"]
dependencies = { mss = "*" }
exposed = { ... }
shortcuts = ["mss"]
```

注意 `shortcuts` 条目。
如果存在，`pixi` 将为 `mss` 包安装快捷方式。
这意味着应用程序将出现在开始菜单中。
如果你想自己打包一个可以从这中受益的应用程序，可以查看相应的[文档](https://conda.github.io/menuinst/)。