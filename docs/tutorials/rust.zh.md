# Rust 教程

在本教程中，我们将向你展示如何使用 `pixi` 开发 Rust 包。
本教程是从上到下执行的，缺少步骤可能会导致错误。

本教程的受众是熟悉 Rust 和 `cargo` 并有兴趣尝试 Pixi 进行开发的开发者。
好处是在 rust 工作流中，你可以同时锁定 rust 和 C/System 依赖项。例如，tokio 用户可能依赖 linux 上的 `openssl`。

## 前提条件

- 你需要已安装 `pixi`。如果你还没有安装，可以按照[安装指南](../index.md)中的说明进行操作。
  本教程的关键是向你展示你只需要 pixi！

## 创建 Pixi workspace

```shell
pixi init my_rust_project
cd my_rust_project
```

它应该创建如下目录结构：

```shell
my_rust_project
├── .gitattributes
├── .gitignore
└── pixi.toml
```

`pixi.toml` 文件是你 workspace 的 manifest 文件。它应该如下所示：

```toml  title="pixi.toml"
[workspace]
name = "my_rust_project"
version = "0.1.0"
description = "Add a short description here"
authors = ["User Name <user.name@email.url>"]
channels = ["conda-forge"]
platforms = ["linux-64"] # (1)!

[tasks]

[dependencies]
```

1. `platforms` 默认设置为你的系统平台。你可以将其更改为你希望支持的任何平台。例如 `["linux-64", "osx-64", "osx-arm64", "win-64"]`。

## 添加 Rust 依赖

使用 Pixi workspace，你不需要系统上的任何依赖，你需要的所有依赖都应该通过 pixi 添加，这样其他用户可以无任何问题地使用你的 workspace。
```shell
pixi add rust
```

这将把 `rust` 包添加到你的 `pixi.toml` 文件的 `[dependencies]` 中。
其中包括 `rust` 工具链和 `cargo`。

## 添加 `cargo` 项目
现在你已经安装了 rust，你可以在你的 `pixi` workspace 中创建一个 `cargo` 项目。
```shell
pixi run cargo init
```

`pixi run` 是 pixi 在环境中运行命令的方式。它将确保为命令运行激活环境。
它运行自己的跨平台 shell，如果你想了解更多，请查看[`任务`文档](../workspace/advanced_tasks.md)。
你也可以通过运行 `pixi shell` 激活环境中的 shell，之后你不再需要 `pixi run ...`。

现在我们可以使用 `pixi` 构建 `cargo` 项目。
```shell
pixi run cargo build
```
为了简化构建过程，你可以使用以下命令将 `build` 任务添加到你的 `pixi.toml` 文件：
```shell
pixi task add build "cargo build"
```
这将在 `pixi.toml` 文件中创建这个字段：
```toml title="pixi.toml"
[tasks]
build = "cargo build"
```

现在你可以使用以下命令构建你的项目：
```shell
pixi run build
```

你也可以使用以下命令运行你的项目：
```shell
pixi run cargo run
```
你可以再次使用任务简化它。
```shell
pixi task add start "cargo run"
```

所以你应该得到以下输出：
```shell
pixi run start
Hello, world!
```

恭喜，你有一个 Rust 项目在机器上使用 pixi 运行！

## 下一步，为什么这在有 `rustup` 时有用？
Cargo 不是二进制包管理器，而是基于源的包管理器。
这意味着你需要在系统上安装 Rust 编译器才能使用它。
可能还需要 `cargo` 包管理器中未包含的其他依赖项。
例如，你可能需要在系统上安装 `openssl` 或 `libssl-dev` 来构建包。
`pixi` 也是这种情况，但 `pixi` 会将这些依赖项安装到你的 workspace 文件夹中，所以你不必担心它们。

将以下依赖项添加到你的 cargo 项目：
```shell
pixi run cargo add git2
```

如果你的系统未预配置为构建 C 且未安装 `libssl-dev` 包，你将无法构建项目：
```shell
pixi run build
...
Could not find directory of OpenSSL installation, and this `-sys` crate cannot
proceed without this knowledge. If OpenSSL is installed and this crate had
trouble finding it,  you can set the `OPENSSL_DIR` environment variable for the
compilation process.

Make sure you also have the development packages of openssl installed.
For example, `libssl-dev` on Ubuntu or `openssl-devel` on Fedora.

If you are in a situation where you think the directory *should* be found
automatically, please open a bug at https://github.com/sfackler/rust-openssl
and include information about your system as well as this message.

$HOST = x86_64-unknown-linux-gnu
$TARGET = x86_64-unknown-linux-gnu
openssl-sys = 0.9.102


It looks like you are compiling on Linux and also targeting Linux. Currently this
requires the `pkg-config` utility to find OpenSSL but unfortunately `pkg-config`
could not be found. If you have OpenSSL installed you can likely fix this by
installing `pkg-config`.
...
```
你可以通过使用 pixi 添加构建 git2 所需的必要依赖项来修复这个问题：
```shell
pixi add openssl pkg-config compilers
```

现在你应该能够再次构建你的项目：
```shell
pixi run build
...
   Compiling git2 v0.18.3
   Compiling my_rust_project v0.1.0 (/my_rust_project)
    Finished dev [unoptimized + debuginfo] target(s) in 7.44s
     Running `target/debug/my_rust_project
```

## 额外：添加更多任务
你可以将更多任务添加到你的 `pixi.toml` 文件以简化你的工作流。

例如，你可以添加一个 `test` 任务来运行你的测试：
```shell
pixi task add test "cargo test"
```

你可以添加一个 `clean` 任务来清理你的项目：
```shell
pixi task add clean "cargo clean"
```

你可以添加一个格式化任务到你的项目：
```shell
pixi task add fmt "cargo fmt"
```

你可以使用 `depends-on` 字段扩展这些任务来运行多个命令：
```shell
pixi task add lint "cargo clippy" --depends-on fmt
```

## 结论
在本教程中，我们向你展示了如何使用 `pixi` 创建 Rust 项目。
我们还向你展示了如何使用 `pixi` **添加依赖**到你的项目。
这样你可以确保你的项目在**任何安装了 `pixi` 的系统**上都是**可复现**的。

## 展示你的作品！
完成你的项目了吗？
我们很想看到你创造了什么！
在社交媒体上使用 #pixi 标签分享你的作品并 @prefix_dev。
让我们一起激励社区！