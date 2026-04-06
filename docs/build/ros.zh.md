本指南展示如何使用 Pixi 的 `pixi-build-ros` 后端将 ROS 包构建为 conda 包。

要了解构建功能，请从常规[构建入门](./getting_started.md)指南开始。
有关不使用 Pixi 构建的 ROS（不是打包），请参阅 [ROS 2 教程](../tutorials/ros2.md)。
您可能还想阅读 [pixi-build-ros](backends/pixi-build-ros.md) 的后端文档。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    预计会有一些不完善之处；请报告问题以便我们改进。

## 创建 Pixi 工作区（workspace）

初始化一个新工作区（workspace）并安装 ROS 2 CLI，以便您可以通过 `ros2` CLI 搭建包。

```bash
pixi init ros_ws --channel https://prefix.dev/robostack-jazzy --channel https://prefix.dev/conda-forge
cd ros_ws
pixi add ros-jazzy-ros2run
```

这将 `ros2` CLI 命令添加到您的 Pixi 环境。

在下面的所有示例中，确保在您的工作区（workspace）清单中启用了[构建预览](../reference/pixi_manifest.md#preview-features)：
```toml title="ros_ws/pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/ros_ws/pixi.toml:preview"
```

生成的工作区（workspace）清单：
```toml title="ros_ws/pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/ros_ws/pixi.toml:base"
```

## 创建 Python ROS 包
我们将创建一个使用 `ament_python` 的普通 ROS2 包，然后为其添加 Pixi 支持。
大部分逻辑由 ROS2 CLI 完成，因此您可以按照常规 ROS 2 包创建步骤进行。

### 初始化 ROS 包

使用 ROS CLI 在工作区（workspace）中生成一个 `ament_python` 包骨架。

```bash
pixi run ros2 pkg create --build-type ament_python --destination-directory src --node-name my_python_node my_python_ros_pkg
```

现在您应该得到类似以下内容：

```
ros_ws/
├── pixi.toml
└── src/
    └── my_python_ros_pkg/
        ├── package.xml
        ├── resource/
        ├── setup.cfg
        ├── setup.py
        ├── test/
        └── my_python_ros_pkg/
            ├── __init__.py
            └── my_python_node.py
```

### 向新包添加 Pixi 包信息

在 `src/my_python_ros_pkg` 内创建 `pixi.toml`，以便 Pixi 可以使用 ROS 后端构建它。后端从 `package.xml` 读取大部分元数据，因此您只需要指定后端和发行版（distro）。

```toml title="src/my_python_ros_pkg/pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/ros_ws/src/my_python_ros_pkg/pixi.toml"
```

注意事项：

- 当设置 `package.build.config.distro` 时，生成的包名称会添加前缀，如 `ros-<distro>-<name>`
- 后端自动读取 `package.xml`（名称、版本、许可证、维护者、URL、依赖）。在 `pixi.toml` 中明确设置的任何字段都会覆盖 `package.xml`
- `package.xml` 中的依赖通过 RoboStack 映射到 conda 包（例如 `std_msgs` → `ros-<distro>-std-msgs`）。未知依赖保持不变

### 将包添加到 pixi 工作区（workspace）

通过与 ROS 前缀名称匹配的路径依赖告诉根工作区（workspace）依赖该包：

```toml title="ros_ws/pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/ros_ws/pixi.toml:python-pkg"
```

### 测试您的包
现在安装并运行：

```bash
pixi run ros2 run my_python_ros_pkg my_python_node
```
输出：
```bash
Hi from my_python_ros_pkg.
```

## 创建 CMake ROS 包

创建使用 `ament_cmake` 的 C++ 或混合包。

### 搭建 C++ 包：

```bash
pixi run ros2 pkg create --build-type ament_cmake --destination-directory src --node-name my_cmake_node my_cmake_ros_pkg
```

### 添加 pixi 包信息

在 `src/my_cmake_ros_pkg` 内创建 `pixi.toml`，以便 Pixi 可以使用 ROS 后端构建它。
后端从 `package.xml` 读取大部分元数据，因此您只需要指定 `backend` 和 `distro`。

```toml title="src/my_cmake_ros_pkg/pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/ros_ws/src/my_cmake_ros_pkg/pixi.toml"
```

### 将包添加到 pixi 工作区（workspace）

通过与 ROS 前缀名称匹配的路径依赖告诉根工作区（workspace）依赖该包：

```toml title="ros_ws/pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/ros_ws/pixi.toml:cmake-pkg"
```

### 测试您的包
现在安装并运行：
```bash
pixi run ros2 run my_cmake_ros_pkg my_cmake_node
```
输出：
```bash
hello world my_cmake_ros_pkg package
```

## 构建 ROS conda 包
将包添加到工作区（workspace）后，您现在可以构建它们。

```bash
cd src/my_python_ros_pkg
pixi build
# 然后
cd ../my_cmake_ros_pkg
pixi build
```

现在您可以将这些产物上传到 conda 通道（channel）并从其他 Pixi 工作区（workspace）依赖它们。

## 技巧和注意事项

- ROS 发行版和平台：选择正确的 RoboStack 通道（channel）（例如 `robostack-humble`、`robostack-jazzy`）并确保您的平台受支持
- 保持 `package.xml` 准确：名称、版本、许可证、维护者、URL 和依赖会自动读取；但您可以在 [pixi 清单](https://pixi.sh/latest/reference/pixi_manifest/#the-package-section) 中覆盖它们
- 后端文档：有关 `env`、`distro` 和 `extra-input-globs` 等配置详情，请参阅 [pixi-build-ros 文档](backends/pixi-build-ros.md)
- Colcon 与 pixi build：使用 `pixi` 时不需要 `colcon`；后端调用正确的构建流程。但因为您不需要更改包结构，如果您愿意，仍然可以使用 `colcon`
- 并非所有 ROS 包都可在 RoboStack 中使用。如果您依赖的包不在 RoboStack 中，您可以：
  - **推荐：** 向 RoboStack 贡献添加它；请参阅 [RoboStack 贡献页面](https://robostack.github.io/Contributing.html)
  - 使用 Pixi 在单独的工作区（workspace）中自行打包并上传到您自己的 conda 通道（channel）
    - （可选）这可以使用[树外包定义](./package_source.md)来构建包，而无需更改其源代码

## 结论

您可以使用 `pixi-build-ros` 后端将 ROS 项目打包为 conda 包。
从简单开始，保持 `package.xml` 准确，根据需要添加 ROS 依赖，并使用预览构建功能进行迭代。
构建完成后，您可以将产物上传到 conda 通道（channel）并从其他 Pixi 工作区（workspace）依赖它们。
