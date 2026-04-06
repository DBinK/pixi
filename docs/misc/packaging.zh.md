这是为想要将 Pixi 打包到不同包管理器中的分发维护者提供的指南。Pixi 用户可以忽略此页面。

## 构建

Pixi 是用 Rust 编写的，使用 Cargo 编译，编译时需要这些依赖。运行时 Pixi 不需要除它编译所针对的运行时（`libc` 等）之外的任何依赖。

要构建 Pixi，请运行：
```shell
cargo build --locked --profile dist
```
除了使用为二进制大小优化的预定义 `dist` 配置文件，你还可以传递其他选项让 cargo 为其他指标优化二进制文件。

### 构建时选项

Pixi 提供一些可以影响构建的编译时选项。

#### TLS

默认情况下，Pixi 使用 Rustls TLS 实现构建。你可以通过在构建命令中添加 `--no-default-feature --feature native-tls` 使用平台原生 TLS 实现编译 Pixi。请注意，这可能会增加额外的运行时依赖，例如 Linux 上的 OpenSSL。

#### 自我更新

Pixi 有自我更新功能。当 Pixi 使用另一个包管理器安装时，通常不希望 pixi 尝试更新自己，而是让它由包管理器更新。
因此，默认情况下禁用自我更新功能。可以通过在构建命令中添加 `--feature self_update` 来启用。

当自我更新功能被禁用且用户尝试运行 `pixi self-update` 时，会显示错误消息。可以通过在构建时将 `PIXI_SELF_UPDATE_DISABLED_MESSAGE` 环境变量设置为指向用户应该用来更新 pixi 的包管理器来自定义此消息。
```shell
PIXISELFUPDATE_DISABLED_MESSAGE="`self-update` has been disabled for this build. Run `brew upgrade pixi` instead" cargo build --locked --profile dist
```

#### 自定义版本

你可以通过在构建时设置 `PIXI_VERSION` 环境变量来指定用于 `--version` 输出的自定义版本字符串。

```shell
PIXIVERSION="HEAD-123456" cargo build --locked --profile dist
```

## Shell 补全

构建 Pixi 后，你可以通过运行：
```shell
pixi completion --shell <SHELL>
```
生成 shell 自动补全脚本并保存到文件。
目前支持的 shell 有 `bash`、`elvish`、`fish`、`nushell`、`powershell` 和 `zsh`。
