# pixi-build-ros

`pixi-build-ros` 后端专为使用原生 ROS 构建系统构建 [ROS（机器人操作系统）](https://www.ros.org/) 包而设计。
不再需要使用 `colcon` 或 `catkin_tools` 来构建您的 ROS 包。
它提供与 Pixi 包管理工作流程的无缝集成，同时支持 ROS1 和 ROS2 包，并具有自动依赖项解析功能。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    这就是为什么我们要求用户通过将 "pixi-build" 添加到 `workspace.preview` 来选择加入该功能。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## 概述

此后端通过以下方式自动从 ROS 项目生成 conda 包：

- **package.xml 集成**：自动从您的 ROS `package.xml` 文件读取包元数据（名称、版本、描述、维护者、依赖项）
- **多构建系统支持**：支持 ament_cmake、ament_python、catkin 和 cmake 构建类型
- **ROS 发行版支持**：支持 ROS1 和 ROS2 发行版（noetic、humble、jazzy 等）
- **跨平台支持**：支持 Linux、macOS 和 Windows
- **自动依赖项映射**：使用 RoboStack 映射将 ROS 依赖项映射到 conda 包

## 基本用法

要在您的 `pixi.toml` 中使用 ROS 后端，请将其添加到包的构建配置中：

```toml
[workspace]
preview = ["pixi-build"]
channels = [
    "https://prefix.dev/robostack-jazzy",  # 或 robostack-humble、robostack-noetic 等
    "https://prefix.dev/conda-forge"
]
platforms = ["linux-64", "osx-arm64"]

[package.build]
backend = { name = "pixi-build-ros", version = "*" }

[package.build.config]
distro = "jazzy"  # 或 "humble"、"noetic" 等
```

??? Note "工作区配置"
    工作区（workspace）可以在包的 `pixi.toml` 中定义，也可以在工作区（workspace）根目录的单独 `pixi.toml` 中定义。
    例如，使用这样的工作区（workspace）结构：
    ```shell
    tree -L 2
    .
    ├── pixi.toml  # 工作区（workspace）配置
    └── src
        └── my_ros_package
            ├── package.xml  # ROS 包清单（manifest）
            └── pixi.toml  # 包配置
    ```

然后您可以运行 `pixi build` 为您的 ROS 包创建 conda 包。
```shell
pixi build
```

当您想将其安装到您的环境时，您可以通过以下方式添加到您的工作区（workspace）`pixi.toml`：

```toml
[dependencies]
ros-jazzy-my-ros-package = { path = "." }
# 或者如果包在单独的 pixi.toml 中
# ros-jazzy-my-ros-package = { path = "src/my_ros_package" }
```
请注意，当您使用发行版配置时，您需要指定 `ros-jazzy-` 前缀。

### 自动元数据检测

后端将自动从您的 `package.xml` 文件中读取元数据，以填充您 `pixi.toml` 中**未**明确定义的包信息。
这包括：

- **包名称和版本**：如果未在 `pixi.toml` 中指定，则自动使用
- **描述**：使用 `package.xml` 中的描述
- **维护者**：从 `package.xml` 中的维护者字段提取
- **主页**：从 `package.xml` 中类型为 "website" 的 URL 字段获取
- **仓库**：从 `package.xml` 中类型为 "repository" 的 URL 字段获取

```xml
<package format="3">
  <name>my_ros_package</name>
  <version>1.0.0</version>
  <description>A useful ROS package for navigation</description>
  <maintainer email="developer@example.com">John Doe</maintainer>
  <url type="website">https://github.com/user/my_ros_package</url>
  <url type="repository">https://github.com/user/my_ros_package</url>
</package>
```

它相当于以下 `pixi.toml`：

```toml
[package]
name = "my_ros_package"
version = "1.0.0"
description = "A useful ROS package for navigation"
maintainers = ["John Doe <developer@example.com"]
homepage = "https://github.com/user/my_ros_package"
repository = "https://github.com/user/my_ros_package"
```

后端将自动使用 `package.xml` 中的元数据生成名为 `ros-jazzy-my-ros-package` 的完整 conda 包。
如果 `pixi.toml` 中明确设置了字段，这些值将覆盖 `package.xml` 中的值。

### 自动发行版检测
`pixi-build-ros` 将根据工作区（workspace）中的 `channels` 自动检测 ROS 发行版。
如果 `pixi.toml` 中未指定 `distro`，它将根据工作区（workspace）中的 `channels` 自动检测。

```toml title="pixi.toml"
[workspace]
channels = ["conda-forge", "robostack-jazzy"]


[package.build.config]
  # 这已经由后端中的函数自动检测。
  # 因为它搜索 'robostack-' 并使用第一个匹配项，如果不是这样定义的话。
distro = "jazzy"
```

实现这一点是为了通过更改 `workspace` 部分中使用的 `channel`，可以轻松地在 ros 包之间切换发行版。

这不适用于 `robostack-staging` 通道（channel），因为它包含多个发行版的包。

### 自动依赖项解析

因为 `package.xml` 文件中的依赖项定义与 conda 包名称不完全相同，后端需要将 ROS 依赖项映射到 conda 包。

- **已知依赖项**：使用 [`robostack.yaml`](https://github.com/prefix-dev/pixi-build-backends/blob/main/backends/pixi-build-ros/robostack.yaml) 进行映射
- **自定义映射**：您可以在 `pixi.toml` 中的[`[package.build.config.extra-package-mappings]`](#extra-package-mappings)提供额外的映射
- **其他包**：映射到 `ros-<distro>-<package-name>` 格式

包名称的 `<distro> 部分基于 `distro` 配置自动生成。

## 配置选项

您可以使用 `pixi.toml` 中的 `[package.build.config]` 部分自定义 ROS 后端行为。后端支持以下配置选项：

### `distro`（可选）

- **类型**: `String`
- **默认值**: 基于工作区（workspace）中的 `channels` 使用[自动检测](#automatic-distro-detection)
- **目标合并行为**: `Overwrite` - 平台特定的发行版优先于基础设置

要构建的 ROS 发行版。这会影响依赖项映射和构建配置。
如果已设置，包名称将自动添加 `ros-<distro>-` 前缀，否则使用 `pixi.toml` 或 `package.xml` 中的包名称。

```toml
[package.build.config]
distro = "jazzy"  # 或 "humble"、"noetic"、"iron" 等
```

### `env`

- **类型**: `Map<String, String>`
- **默认值**: `{}`
- **目标合并行为**: `Merge` - 平台环境变量用相同名称覆盖基础变量，其他变量合并

构建过程中设置的环境变量。这些变量在编译期间可用。

```toml
[package.build.config]
env = { AMENT_CMAKE_ENVIRONMENT_HOOKS_ENABLED = "1" }
```

#### 自动注入的环境变量

ROS 后端使以下变量与选定的发行版保持同步，因此您不需要手动在 `env` 中设置它们：

- `ROS_DISTRO` — 设置为您在 `distro` 中配置的发行版名称
- `ROS_VERSION` — 对于 ROS 1 发行版设置为 `"1"`，对于 ROS 2 发行版设置为 `"2"`

这些值在评估 `package.xml` 条件期间和生成的构建脚本中都可用。您在 `env` 中提供的任何自定义条目都会合并在这些默认值之上。
如果您在 `env` 中显式设置 `ROS_DISTRO` 或 `ROS_VERSION`，您的值将优先于默认值。

### `debug-dir`

后端始终将 JSON-RPC 请求/响应日志和生成的中间食谱写入工作目录内的 `debug` 子目录（例如 `<work_directory>/debug`）。已弃用的 `debug-dir` 配置选项将被忽略；如果它仍在清单中设置，后端会发出警告，以便您可以安全地删除它。

### `extra-input-globs`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定 glob 完全替换基础 glob

作为构建过程的输入文件包含的额外 glob 模式。这些模式被添加到包含 ROS 相关文件的默认输入 glob 中。

```toml title="pixi.toml"
[package.build.config]
extra-input-globs = [
    "launch/**/*.py",
    "config/*.yaml",
    "msgs/**/*.msg",
    "srvs/**/*.srv"
]
```

默认输入 glob 包括：
- 源文件：`**/*.{c,cpp,h,hpp,rs,sh,py,pyx}`
- ROS 配置：`package.xml`、`setup.py`、`setup.cfg`、`pyproject.toml`
- 构建文件：`CMakeLists.txt`

### `extra-package-mappings`

- **类型**: `List<Map<String, Map<String, List<String> | RelativeFileName>>>`
- **默认值**: `[]`

要应用于依赖项映射过程的额外依赖项映射。这些映射用于扩展 `package.xml` 文件中依赖项的使用。

```toml title="pixi.toml"
[package.build.config]
extra-package-mappings = [
    {"ros-custom" = { ros =  ["ros-custom-msgs"] }},
    "mapping.yml"
]
```

或者使用 toml 表数组：

```toml title="pixi.toml"
[[package.build.config.extra-package-mappings]]
custom_msgs = { ros = ["custom-messages"] }
```

或者您可以直接在列表中使用文件：

```toml title="pixi.toml"
[package.build.config]
extra-package-mappings = ["mapping.yml"]
```

映射文件可以包含以下内容：

```yaml title="mapping.yml"
package_name:  # package.xml 中的包名称
  conda: conda-package-name # 映射到 conda 包，例如来自 `conda-forge`
package_name2: # package.xml 中的包名称
  conda: [package1, package2] # 映射到 conda 包列表
ros_package:   # package.xml 中的包名称
  ros: ros_package # 映射到 RoboStack 风格的包名称，例如 `ros-<distro>-ros-package`
```

## 默认变体

在 Windows 平台上，后端自动设置以下默认变体：

- `c_compiler`: `vs2022` - Visual Studio 2022 C 编译器
- `cxx_compiler`: `vs2022` - Visual Studio 2022 C++ 编译器

设置此默认值是为了与 conda-forge 切换到 Visual Studio 2022 保持一致，因为[主流支持 Visual Studio 2019 已 于 2024 年结束](https://learn.microsoft.com/en-us/lifecycle/products/visual-studio-2019)。
`vs2022` 编译器在现代 GitHub 运行器和构建环境中得到更广泛的支持。

您可以通过在 `pixi.toml` 中使用[`[workspace.build-variants]`](https://pixi.sh/latest/reference/pixi_manifest/#build-variants-optional)显式设置变体来覆盖这些默认值：

```toml
[workspace.build-variants]
c_compiler = ["vs2019"]
cxx_compiler = ["vs2019"]
```

## 构建流程

ROS 后端遵循以下构建流程：

1. **包检测**：解析 `package.xml` 以确定构建类型（`ament_cmake`、`ament_python`、`catkin`）
2. **依赖项解析**：使用 RoboStack 映射将 ROS 依赖项映射到 conda 包
3. **环境设置**：配置 ROS 特定的环境变量
4. **构建执行**：根据包类型使用适当的构建模板
5. **安装**：将构建的工件安装到 conda 包环境前缀（prefix）

## ROS 包类型

后端支持不同的 ROS 包构建类型：

### ament_cmake (ROS2)
对于使用 ament 构建系统的 C++ 包：
```xml
<export>
  <build_type>ament_cmake</build_type>
</export>
```

### ament_python (ROS2)
对于使用 ament 构建系统的 Python 包：
```xml
<export>
  <build_type>ament_python</build_type>
</export>
```

### catkin (ROS1)
对于使用 catkin 构建系统的 ROS1 包：
```xml
<export>
  <build_type>catkin</build_type>
</export>
```

## 限制

- **版本约束**：目前忽略来自 `package.xml` 的依赖项版本
- **条件依赖项**：尚不完全支持带条件的依赖项
- **特定于目标的依赖项**：`package.xml` 中的平台特定依赖项需要手动处理

## 另请参阅

- [ROS 文档](https://docs.ros.org/) - 官方 ROS 文档
- [RoboStack](https://robostack.github.io/) - 机器人操作系统的 Conda 包
- [ament 构建系统](https://docs.ros.org/en/rolling/Concepts/Build-System-Development/ament.html) - ROS2 构建系统
- [catkin 构建系统](http://wiki.ros.org/catkin) - ROS1 构建系统
