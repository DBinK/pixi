# ROS 2 教程

在本教程中，我们将向你展示如何使用 `pixi` 开发 ROS 2 包。
本教程是从上到下执行的，缺少步骤可能会导致错误。

本教程的受众是熟悉 ROS 2 并有兴趣尝试 Pixi 进行开发的开发者。

## 前提条件

- 你需要已安装 `pixi`。如果你还没有安装，可以按照[安装指南](../index.md)中的说明进行操作。
  本教程的关键是向你展示你只需要 pixi！
- 在 Windows 上，建议启用开发者模式。转到设置 -> 更新和安全 -> 开发者 -> 开发者模式。

## 创建 Pixi workspace

```shell
pixi init my_ros2_project -c robostack-humble -c conda-forge
cd my_ros2_project
```

它应该创建如下目录结构：

```shell
my_ros2_project
├── .gitattributes
├── .gitignore
└── pixi.toml
```

`pixi.toml` 文件是你 workspace 的 manifest 文件。它应该如下所示：

```toml title="pixi.toml"
[workspace]
name = "my_ros2_project"
version = "0.1.0"
description = "Add a short description here"
authors = ["User Name <user.name@email.url>"]
channels = ["robostack-humble", "conda-forge"]
# Your project can support multiple platforms, the current platform will be automatically added.
platforms = ["linux-64"]

[tasks]

[dependencies]
```

你在 `init` 命令中添加的 `channels` 是包仓库，你可以通过我们的 [prefix.dev](https://prefix.dev/channels) 网站搜索这些仓库。
`platforms` 是你希望支持的系统，在 Pixi 中你可以支持多个平台，但你必须定义哪些平台，这样 Pixi 可以测试你的依赖是否支持这些平台。
对于其余字段，你可以根据需要进行填写。

## 添加 ROS 2 依赖

使用 Pixi workspace，你不需要系统上的任何依赖，你需要的所有依赖都应该通过 pixi 添加，这样其他用户可以无任何问题地使用你的 workspace。

让我们从 `turtlesim` 示例开始

```shell
pixi add ros-humble-desktop ros-humble-turtlesim
```

这将把 `ros-humble-desktop` 和 `ros-humble-turtlesim` 包添加到你的 manifest 中。
根据你的网络速度，这可能需要一分钟，因为它还会在你的 workspace 文件夹（`.pixi`）中安装 ROS。

现在运行 `turtlesim` 示例。

```shell
pixi run ros2 run turtlesim turtlesim_node
```

**或者**使用 `shell` 命令在终端中激活的环境中启动 shell。

```shell
pixi shell
ros2 run turtlesim turtlesim_node
```

恭喜，你已经在机器上使用 pixi 运行了 ROS 2！

??? example "与乌龟的更多玩法"
    要控制乌龟，你可以在新终端中运行以下命令
    ```shell
    cd my_ros2_project
    pixi run ros2 run turtlesim turtle_teleop_key
    ```
    现在你可以用键盘上的方向键控制乌龟了。

![Turtlesim control](https://github.com/user-attachments/assets/9424c44b-b7c0-48f4-8e7d-501131e9e9e5)

## 添加自定义 Python 节点

由于 ros 使用自定义节点，让我们为我们的项目添加一个自定义节点。

```shell
pixi run ros2 pkg create --build-type ament_python --destination-directory src --node-name my_node my_package
```

要构建包，我们需要一些更多的依赖：

```shell
pixi add colcon-common-extensions "setuptools<=58.2.0"
```

将创建的 ros workspace 初始化脚本添加到你的 manifest 文件中。

然后运行构建命令

```shell
pixi run colcon build
```

这将在 `install` 文件夹中创建一个可获取的脚本，你可以通过激活脚本使用你的自定义节点。
通常这会是你添加到 `.bashrc` 的脚本，但你告诉 Pixi 通过在 `pixi.toml` 中添加以下内容来使用它：

=== "Linux & macOS"
    ```toml title="pixi.toml"
    [activation]
    scripts = ["install/setup.sh"]
    ```

=== "Windows"
    ```toml title="pixi.toml"
    [activation]
    scripts = ["install/setup.bat"]
    ```

??? tip "多平台支持"
    你可以为不同的平台添加多个激活脚本，这样你可以通过一个 workspace 支持多个平台。
    使用以下示例添加对 Linux 和 Windows 的支持，使用 [target](../workspace/multi_platform_configuration.md#activation) 语法。

    ```toml
    [workspace]
    platforms = ["linux-64", "win-64"]

    [activation]
    scripts = ["install/setup.sh"]
    [target.win-64.activation]
    scripts = ["install/setup.bat"]
    ```

现在你可以用以下命令运行你的自定义节点

```shell
pixi run ros2 run my_package my_node
```

## 简化用户体验

在 `pixi` 中，我们有一个名为 `tasks` 的功能，它允许你在 manifest 文件中定义任务，并通过简单命令运行它。
让我们添加一个任务来运行 `turtlesim` 示例和自定义节点。

```shell
pixi task add sim "ros2 run turtlesim turtlesim_node"
pixi task add build "colcon build --symlink-install"
pixi task add hello "ros2 run my_package my_node"
```

现在你可以通过简单运行来运行这些任务

```shell
pixi run sim
pixi run build
pixi run hello
```

???+ tip "高级任务用法"
    任务是 pixi 中的强大功能。

    - 你可以向任务添加 [`depends-on`](../workspace/advanced_tasks.md#depends-on) 来创建任务链。
    - 你可以向任务添加 [`cwd`](../workspace/advanced_tasks.md#working-directory) 来在不同目录中运行任务。
    - 你可以向任务添加 [`inputs` 和 `outputs`](../workspace/advanced_tasks.md#caching) 来创建仅在输入更改时运行的任务。
    - 你可以使用 [`target`](../reference/pixi_manifest.md#the-target-table) 语法在特定机器上运行特定任务。

```toml
[tasks]
sim = "ros2 run turtlesim turtlesim_node"
build = {cmd = "colcon build --symlink-install", inputs = ["src"]}
hello = { cmd = "ros2 run my_package my_node", depends-on = ["build"] }
```

## 构建 C++ 节点

要构建 C++ 节点，你需要将 `ament_cmake` 和一些其他构建依赖添加到你的 manifest 文件中。

```shell
pixi add ros-humble-ament-cmake-auto compilers pkg-config "cmake<4" ninja colcon-common-extensions
```

现在你可以用以下命令创建 C++ 节点

```shell
pixi run ros2 pkg create --build-type ament_cmake --destination-directory src --node-name my_cpp_node my_cpp_package
```

现在你可以再次构建并用以下命令运行它

```shell
# Passing arguments to the build command to build with Ninja, add them to the manifest if you want to default to ninja.
pixi run build --cmake-args -G Ninja
pixi run ros2 run my_cpp_package my_cpp_node
```

??? tip
    将 cpp 任务添加到 manifest 文件以简化用户体验。

    ```shell
    pixi task add hello-cpp "ros2 run my_cpp_package my_cpp_node"
    ```

## 结论

在本教程中，我们向你展示了如何使用 `pixi` 创建 Python 和 CMake ROS2 项目。
我们还向你展示了如何使用 `pixi` **添加依赖**到你的项目，以及如何使用 `pixi run` **运行你的项目**。
这样你可以确保你的项目在所有安装了 `pixi` 的机器上都是**可复现**的。

## 展示你的作品！

完成你的项目了吗？
我们很想看到你创造了什么！
在社交媒体上使用 #pixi 标签分享你的作品并 @prefix_dev。
让我们一起激励社区！

## 常见问题

### `rosdep` 会怎么样？

目前，我们不支持 Pixi 环境中的 `rosdep`，所以你必须使用 `pixi add` 添加包。
`rosdep` 会调用 `conda install`，这在 Pixi 环境中不支持。

### 我可以在哪里找到更多关于 `robostack-*` channels 的文档？

你可以在 [RoboStack 文档](https://robostack.github.io/) 中找到更多关于 RoboStack channels 的文档。

### 社区示例

ROS 2 Humble on macOS，[使用 Gazebo 模拟差分驱动](https://medium.com/@davisogunsina/ros-2-macos-support-installing-and-running-ros-2-on-macos-79039d1d3655)。