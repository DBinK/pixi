# 安装

在终端中运行以下命令安装 `pixi`：

=== "Linux & macOS"
    ```bash
    curl -fsSL https://pixi.sh/install.sh | sh
    ```

    如果你的系统没有 `curl`，可以使用 `wget`：

    ```bash
    wget -qO- https://pixi.sh/install.sh | sh
    ```

    ??? note "这是做什么的？"
        上述命令会自动下载最新版本的 `pixi`，解压并将其二进制文件移动到 `~/.pixi/bin`。
        脚本还会将 `PATH` 环境变量扩展到 shell 启动脚本中，以包含 `~/.pixi/bin`。
        这样你就可以从任何地方调用 `pixi`。

=== "Windows"
    [下载安装程序](https://github.com/prefix-dev/pixi/releases/latest/download/pixi-x86_64-pc-windows-msvc.msi){ .md-button }

    或者运行：

    ```powershell
    powershell -ExecutionPolicy Bypass -c "irm -useb https://pixi.sh/install.ps1 | iex"
    ```

    ??? note "这是做什么的？"
        上述命令会自动下载最新版本的 `pixi`，解压并将其二进制文件移动到 `%UserProfile%\.pixi\bin`。
        该命令还会将 `%UserProfile%\.pixi\bin` 添加到你的 `PATH` 环境变量中，允许你从任何地方调用 `pixi`。

现在重启终端或 shell 以使安装生效。

??? question "不信任我们的链接？检查脚本！"
    你可以查看安装 `sh` 脚本：[下载](https://pixi.sh/install.sh) 和 `ps1`：[下载](https://pixi.sh/install.ps1)。
    这些脚本是开源的，可在 [GitHub](https://github.com/prefix-dev/pixi/tree/main/install) 上获取。

!!! note "别忘了添加自动补全！"
    安装 Pixi 后，你可以为 shell 启用自动补全。
    请参阅下面的[自动补全](#autocompletion)部分获取说明。

## 更新

更新非常简单，重新运行安装脚本即可获取最新版本。

```shell
pixi self-update
```
或使用以下命令获取特定版本：
```shell
pixi self-update --version x.y.z
```

!!! note
    如果你使用包管理器（如 `brew`、`mamba`、`conda`、`paru` 等）安装了 `pixi`，
    你必须使用内置的更新机制。例如 `brew upgrade pixi`。

## 其他安装方式

虽然我们推荐使用上述方法安装 Pixi，但也提供了其他安装方式。

### Homebrew

Pixi 可以通过 homebrew 安装。简单地运行：

```shell
brew install pixi
```

### Windows 安装程序

我们在 [GitHub releases 页面](https://github.com/prefix-dev/pixi/releases/latest) 提供了 `msi` 安装程序。
安装程序会下载 Pixi 并将其添加到 `PATH`。

### Winget

```
winget install prefix-dev.pixi
```

### Scoop

```
scoop install main/pixi
```

### 从 GitHub Releases 下载

Pixi 是一个独立的可执行文件，可以在没有任何外部依赖的情况下运行。
这意味着你可以手动从我们的 [GitHub releases](https://github.com/prefix-dev/pixi/releases) 下载适合你架构和操作系统的存档，解压后直接使用。
如果你希望 `pixi` 本身或通过 `pixi global` 安装的可执行文件在你的 `PATH` 中可用，你需要手动添加它们。
可执行文件位于 [PIXI_HOME](reference/environment_variables.md)/bin。

### 从源码安装

pixi 100% 使用 Rust 编写，因此可以使用 cargo 安装、构建和测试。
要从源码构建开始使用 Pixi，运行：

```shell
cargo install --locked --git https://github.com/prefix-dev/pixi.git pixi
```

我们不再发布到 `crates.io`，因此你需要从仓库安装。
原因是我们依赖一些未发布的 crate，导致无法发布到 `crates.io`。

或者当你想进行修改时使用：

```shell
cargo build
cargo test
```

如果你在构建过程中遇到关于 `rattler` 依赖的问题，请查看其
[编译步骤](https://github.com/conda/rattler/tree/main#give-it-a-try)。

## 安装脚本选项

=== "Linux & macOS"

    安装脚本有多个可以通过环境变量操作的选项。

    | 变量                  | 说明                                                                         | 默认值                    |
    |---------------------|----------------------------------------------------------------------------|-----------------------|
    | `PIXI_VERSION`      | 要安装的 Pixi 版本，可用于升级或降级。                                          | `latest`              |
    | `PIXI_HOME`         | 包含全局环境和配置的 pixi 主文件夹位置。                                       | `$HOME/.pixi`         |
    | `PIXI_BIN_DIR`      | 独立 pixi 二进制文件应安装的位置。                                            | `$PIXI_HOME/bin`      |
    | `PIXI_ARCH`         | Pixi 版本构建的架构。                                                         | `uname -m`            |
    | `PIXI_NO_PATH_UPDATE`| 如果设置，则不会更新 `$PATH` 将 pixi 添加到其中。                           |                       |
    | `PIXI_DOWNLOAD_URL` | 覆盖 Pixi 二进制文件的下载 URL（对镜像或自定义构建很有用）。                     | GitHub releases，如 [linux-64](https://github.com/prefix-dev/pixi/releases/latest/download/pixi-x86_64-unknown-linux-musl.tar.gz)       |
    | `NETRC`             | 用于私有仓库认证的自定义 `.netrc` 文件路径。                                   |                       |
    | `TMP_DIR`           | 脚本用于下载和解压二进制文件的临时目录。                                        | `/tmp`                |

    例如，在 Apple Silicon 上，你可以强制安装 x86 版本：
    ```shell
    curl -fsSL https://pixi.sh/install.sh | PIXI_ARCH=x86_64 bash
    ```
    或设置版本
    ```shell
    curl -fsSL https://pixi.sh/install.sh | PIXI_VERSION=v0.18.0 bash
    ```

    要将 pixi 直接"插入"安装到用户 `$PATH` 中：
    ```shell
    curl -fsSL https://pixi.sh/install.sh | PIXI_BIN_DIR=/usr/local/bin PIXI_NO_PATH_UPDATE=1 bash
    ```

    #### 使用 `.netrc` 进行认证

    如果你需要从需要认证的私有仓库下载 Pixi，可以使用 `.netrc` 文件而不是在 `PIXI_DOWNLOAD_URL` 中硬编码凭证。

    安装脚本自动对 `curl` 和 `wget` 使用 `.netrc` 进行认证。默认情况下，它会查找 `~/.netrc`。你可以使用 `NETRC` 环境变量指定自定义位置：

    ```shell
    # 使用默认的 ~/.netrc 文件
    curl -fsSL https://pixi.sh/install.sh | PIXI_DOWNLOAD_URL=https://private.example.com/pixi-latest.tar.gz bash
    ```

    ```shell
    # 使用自定义 .netrc 文件
    curl -fsSL https://pixi.sh/install.sh | NETRC=/path/to/custom/.netrc PIXI_DOWNLOAD_URL=https://private.example.com/pixi-latest.tar.gz bash
    ```

    你的 `.netrc` 文件应包含以下格式的凭证：
    ```
    machine private.example.com
    login your-username
    password your-token-or-password
    ```

    !!! tip "安全建议"
        使用 `.netrc` 比直接在 `PIXI_DOWNLOAD_URL` 中嵌入凭证更安全（例如 `https://user:pass@example.com/file`），因为它将凭证与 URL 分开，防止它们出现在日志或进程列表中。

    !!! tip "安全说明"
        安装脚本在显示消息时会自动隐藏下载 URL 中嵌入的凭证，将其替换为 `***:***@`，以防止凭证出现在日志或控制台输出中。

=== "Windows"

    安装脚本有多个可以通过环境变量操作的选项。

    | 环境变量             | 说明                                                                       | 默认值                    |
    |---------------------|---------------------------------------------------------------------------|-----------------------------|
    | `PIXI_VERSION`      | 要安装的 Pixi 版本，可用于升级或降级。                                        | `latest`                    |
    | `PIXI_HOME`         | 安装位置。                                                                 | `$Env:USERPROFILE\.pixi`    |
    | `PIXI_NO_PATH_UPDATE`| 如果设置，则不会更新 `$PATH` 将 pixi 添加到其中。                          | `false`                     |
    | `PIXI_DOWNLOAD_URL` | 覆盖 Pixi 二进制文件的下载 URL（对镜像或自定义构建很有用）。                    | GitHub releases，如 [win-64](https://github.com/prefix-dev/pixi/releases/latest/download/pixi-x86_64-pc-windows-msvc.zip)           |

    例如，设置版本：
    ```powershell
    $env:PIXI_VERSION='v0.18.0'; powershell -ExecutionPolicy Bypass -Command "iwr -useb https://pixi.sh/install.ps1 | iex"
    ```

    #### 私有仓库认证

    如果你需要从需要认证的私有仓库下载 Pixi，可以在 `PIXI_DOWNLOAD_URL` 中嵌入凭证。安装脚本会自动在输出中隐藏凭证以确保安全。

    ```powershell
    $env:PIXI_DOWNLOAD_URL='https://username:token@private.example.com/pixi-latest.zip'; powershell -ExecutionPolicy Bypass -Command "iwr -useb https://pixi.sh/install.ps1 | iex"
    ```

    !!! tip "安全说明"
        PowerShell 安装脚本在显示消息时会自动隐藏下载 URL 中嵌入的凭证，将其替换为 `***:***@`，以防止凭证出现在日志或控制台输出中。

## 自动补全

要获取自动补全，请按照你的 shell 的说明进行操作。
之后，重启 shell 或加载 shell 配置文件。

=== "Bash"
    在 `~/.bashrc` 末尾添加以下内容：
    ```bash title="~/.bashrc"
    eval "$(pixi completion --shell bash)"
    ```

=== "Zsh"
    在 `~/.zshrc` 末尾添加以下内容：

    ```zsh title="~/.zshrc"
    autoload -Uz compinit && compinit
    eval "$(pixi completion --shell zsh)"
    ```

=== "PowerShell"
    在 `Microsoft.PowerShell_profile.ps1` 末尾添加以下内容。
    你可以通过在 PowerShell 中查询 `$PROFILE` 变量来检查此文件的位置。
    通常路径是 `~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1` 或
    在 -Nix 上的 `~/.config/powershell/Microsoft.PowerShell_profile.ps1`。

    ```pwsh
    (& pixi completion --shell powershell) | Out-String | Invoke-Expression
    ```

=== "Fish"
    在 `~/.config/fish/config.fish` 末尾添加以下内容：

    ```fish title="~/.config/fish/config.fish"
    pixi completion --shell fish | source
    ```
=== "Nushell"
    在你的 Nushell 配置文件添加以下内容（在 Nushell 中运行 `$nu.config-path` 找到它）：

    ```nushell
    mkdir $"($nu.data-dir)/vendor/autoload"
    pixi completion --shell nushell | save --force $"($nu.data-dir)/vendor/autoload/pixi-completions.nu"
    ```

=== "Elvish"
    在 `~/.elvish/rc.elv` 末尾添加以下内容：

    ```elv title="~/.elvish/rc.elv"
    eval (pixi completion --shell elvish | slurp)
    ```

## 卸载

卸载前，你可能想要删除 pixi 管理的所有文件。

1. 清理任何缓存数据：
    ```shell
    pixi clean cache
    ```
2. 从你的 pixi workspace 中移除环境：
    ```shell
    cd path/to/workspace && pixi clean
    ```
3. 移除 `pixi` 及其全局环境
    ```shell
    rm -r ~/.pixi
    ```
4. 从你的 `PATH` 中移除 pixi 二进制文件：
   - 对于 Linux 和 macOS，在 shell 配置文件中从你的 `PATH` 移除 `~/.pixi/bin`（例如 `~/.bashrc`、`~/.zshrc`）。
   - 对于 Windows，从你的 `PATH` 环境变量中移除 `%UserProfile%\.pixi\bin`。