---
title: 首页
template: home.html
---
# Pixi

Pixi 是一款**快速、现代且可复现**的包管理工具，适用于各种背景的开发者。

## 核心特性

- [🔄 **可复现性**](workspace/lockfile.md)  
  内置锁文件的隔离环境，可轻松重建

- [🛠️ **任务系统**](workspace/advanced_tasks.md)  
  轻松管理复杂的工作流

- [🌐 **多平台支持**](workspace/multi_platform_configuration.md)  
  支持 Linux、macOS、Windows 等系统

- [🧩 **多环境支持**](workspace/multi_environment.md)  
  在一个 manifest 中组合多个环境

- [🐍 **Python 支持**](python/tutorial.md)  
  通过 uv 支持 `pyproject.toml` 和 PyPI

- [💾 **磁盘高效**](workspace/environment.md#de-duplication)
  环境通过硬链接或 reflinks 共享文件，包仅存储一次

- [🌍 **全局工具**](global_tools/introduction.md)
  安全隔离地安装全局工具，替代 `apt`、`homebrew`、`winget`

---

## 快速演示

使用 Pixi 设置项目非常简单。

```shell
pixi init hello-world
cd hello-world
pixi add python
pixi run python -c 'print("Hello World!")'
```

![Pixi 演示](assets/vhs-tapes/pixi_project_demo_light.gif#only-light)
![Pixi 演示](assets/vhs-tapes/pixi_project_demo_dark.gif#only-dark)

安装你喜欢的工具只需一条命令。

```shell
pixi global install gh nvim ipython btop ripgrep
```

![Pixi 全局演示](assets/vhs-tapes/pixi_global_demo_light.gif#only-light)
![Pixi 全局演示](assets/vhs-tapes/pixi_global_demo_dark.gif#only-dark)

---

## 工具对比

| 内置核心功能        | Pixi | Conda | Pip | Poetry | uv |
|-------------------|------|-------|-----|--------|----|
| 安装 Python        | ✅    | ✅     | ❌   | ❌      | ✅  |
| 支持多语言         | ✅    | ✅     | ❌   | ❌      | ❌  |
| 锁文件             | ✅    | ❌     | ❌   | ✅      | ✅  |
| 任务运行器         | ✅    | ❌     | ❌   | ❌      | ❌  |
| workspace 管理    | ✅    | ❌     | ❌   | ✅      | ✅  |

---

## 可用软件

Pixi 默认使用**最大的 Conda 包仓库** [conda-forge](https://conda-forge.org/)，包含超过 **30,000 个包**。

- **Python**: [`python`](https://prefix.dev/channels/conda-forge/packages/python)、[`scikit-learn`](https://prefix.dev/channels/conda-forge/packages/scikit-learn)、[`pytorch`](https://prefix.dev/channels/conda-forge/packages/pytorch)
- **C/C++**: [`clang`](https://prefix.dev/channels/conda-forge/packages/clang)、[`boost`](https://prefix.dev/channels/conda-forge/packages/libboost-devel)、[`opencv`](https://prefix.dev/channels/conda-forge/packages/opencv)、[`ninja`](https://prefix.dev/channels/conda-forge/packages/ninja)
- **Java**: [`openjdk`](https://prefix.dev/channels/conda-forge/packages/openjdk)、[`gradle`](https://prefix.dev/channels/conda-forge/packages/gradle)、[`maven`](https://prefix.dev/channels/conda-forge/packages/maven)
- **Rust**: [`rust`](https://prefix.dev/channels/conda-forge/packages/rust)、[`cargo-edit`](https://prefix.dev/channels/conda-forge/packages/cargo-edit)、[`cargo-insta`](https://prefix.dev/channels/conda-forge/packages/cargo-insta)
- **Node.js**: [`nodejs`](https://prefix.dev/channels/conda-forge/packages/nodejs)、[`bun`](https://prefix.dev/channels/conda-forge/packages/bun)、[`pnpm`](https://prefix.dev/channels/conda-forge/packages/pnpm)、[`eslint`](https://prefix.dev/channels/conda-forge/packages/eslint)
- **命令行工具**: [`git`](https://prefix.dev/channels/conda-forge/packages/git)、[`gh`](https://prefix.dev/channels/conda-forge/packages/gh)、[`ripgrep`](https://prefix.dev/channels/conda-forge/packages/ripgrep)、[`make`](https://prefix.dev/channels/conda-forge/packages/make)

在 [prefix.dev](https://prefix.dev/) 浏览更多软件包，或托管[你自己的 channel](https://prefix.dev/channels/)

---

## 安装

安装 `pixi`，运行：

=== "Linux & macOS"
    ```bash
    curl -fsSL https://pixi.sh/install.sh | sh
    ```

=== "Windows"
    [下载安装程序](https://github.com/prefix-dev/pixi/releases/latest/download/pixi-x86_64-pc-windows-msvc.msi){ .md-button }

    或者运行：

    ```powershell
    powershell -ExecutionPolicy Bypass -c "irm -useb https://pixi.sh/install.ps1 | iex"
    ```

!!! tip "现在重启终端或 shell！"
    安装需要通过重启终端或重新加载 shell 配置来生效。

??? question "不信任我们的链接？检查脚本！"
    你可以查看安装 `sh` 脚本：[下载](https://pixi.sh/install.sh) 和 `ps1`：[下载](https://pixi.sh/install.ps1)。
    这些脚本是开源的，可在 [GitHub](https://github.com/prefix-dev/pixi/tree/main/install) 上获取。

[**查看所有安装选项 →**](installation.md)

---

## 快速入门

=== "Python"
    1. **初始化 workspace：**
        ```
        pixi init hello-world
        cd hello-world
        ```
    2. **添加依赖到默认环境：**
        ```
        pixi add cowpy python
        ```
    3. **创建脚本：**
        ```py title="hello.py"
        --8<-- "docs/source_files/pixi_workspaces/introduction/deps_add/hello.py"
        ```
    5. **添加任务：**
        ```
        pixi task add start python hello.py
        ```
    6. **运行任务：**
        ```
        pixi run start
        ```
        ```
        ✨ Pixi task (start): python hello.py
         __________________
         < Hello Pixi fans! >
         ------------------
              \   ^__^
               \  (oo)\_______
                  (__)\       )\/\
                    ||----w |
                    ||     ||
        ```
    7. **进入环境的 shell：**
        ```
        pixi shell
        python hello.py
        exit
        ```

    更多关于 Pixi 与 Python 配合使用的详情，请参阅 [Python 教程](python/tutorial.md)。

=== "Rust"
    1. **初始化 workspace：**
        ```
        pixi init pixi-rust
        cd pixi-rust
        ```
    2. **添加依赖：**
        ```
        pixi add rust
        ```
    3. **创建 workspace：**
        ```
        pixi run cargo init
        ```
    4. **添加任务：**
        ```
        pixi task add start cargo run
        ```
    5. **运行任务：**
        ```
        pixi run start
        ```
        ```
        ✨ Pixi task (start): cargo run
            Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
             Running `target/debug/pixi-rust`
        Hello, world!
        ```

    这更多是展示 Pixi 与 Rust 配合使用的便捷性。
    不是推荐的 Rust 项目构建方式。
    更多关于 Pixi 与 Rust 配合使用的详情，请参阅 [Rust 教程](tutorials/rust.md)。

=== "Node.js"
    1. **初始化 workspace：**
        ```
        pixi init pixi-node
        cd pixi-node
        ```
    2. **添加依赖：**
        ```
        pixi add nodejs
        ```
    3. **创建脚本：**
        ```js title="hello.js"
        console.log("Hello Pixi fans!");
        ```
    4. **添加任务：**
        ```
        pixi task add start "node hello.js"
        ```
    5. **运行任务：**
        ```
        pixi run start
        ```
        ```
        ✨ Pixi task (start): node hello.js
        Hello Pixi fans!
        ```

=== "ROS2"

    1. **初始化 workspace：**
        ```
        pixi init pixi-ros2 -c https://prefix.dev/conda-forge -c "https://prefix.dev/robostack-humble"
        cd pixi-ros2
        ```
    2. **添加依赖：**
        ```
        pixi add ros-humble-desktop
        ```

        ??? tip "这可能需要一分钟"
            根据你的网络连接，下载整个 ROS2 桌面包会花费较长时间。

    3. **启动 Rviz**
        ```
        pixi run rviz2
        ```

    更多关于 Pixi 与 ROS2 配合使用的详情，请参阅 [ROS2 教程](tutorials/ros2.md)。

=== "DevOps"
    1. 一条命令安装所有喜欢的工具：
    ```shell
    pixi global install terraform ansible k9s make
    ```
    2. 随时随地使用：
    ```shell
    ansible --version
    terraform --version
    k9s version
    make --version
    ```

---

## 开发者评价

<div class="quote-scroll-wrapper">
  <div class="quote-scroll">
    <div class="quote-card">
      <p>"Pixi 是我管理 Python 环境的首选工具。它显著减少了样板代码，因为它同时无缝支持 PyPI 和 conda-forge 索引——这是我工作流程中的关键需求。"</p>
      <strong>Guillaume Lemaitre</strong> – <a href="https://scikit-learn.org">scikit-learn</a>
    </div>
    <div class="quote-card">
      <p>"我无法强调我有多喜欢使用 Pixi global 作为日常 CLI 工具的包管理器。使用 global manifest，即使在多台机器之间共享我的配置也轻而易举！"</p>
      <strong>Matthew Feickert</strong> – <a href="https://www.wisc.edu/">University of Wisconsin–Madison</a>
    </div>
    <div class="quote-card">
      <p>"我们正在改变在 Windows 上管理 ROS 依赖的方式。我们将使用 Pixi 从 conda 安装和管理依赖。对于用户来说，未来这将变得多么简单，我感到非常兴奋。"</p>
      <strong>Michael Carroll</strong> – <a href="https://www.ros.org/">Project Lead ROS</a>
    </div>
  </div>
</div>

---

## 相关链接

- [GitHub](https://github.com/prefix-dev/pixi)：Pixi 源代码，欢迎 star！
- [Discord](https://discord.gg/kKV8ZxyzY4)：加入我们的社区并提问。
- [Prefix.dev](https://prefix.dev/)：支持 Pixi 的公司，构建包管理的未来。
- [conda-forge](https://conda-forge.org/)：社区驱动的 conda 包管理器食谱集合。
- [Rattler](https://github.com/conda/rattler)：用 Rust 重写的 conda 一切。Pixi 的后端。
- [rattler-build](https://rattler.build)：快速的 conda 包构建系统。