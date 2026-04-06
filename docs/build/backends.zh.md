为了将 conda 包的构建与 Pixi 解耦，我们提供了所谓的构建后端。
这些本质上是遵循特定协议的可执行文件，为 Pixi 和构建后端实现。
这还允许将构建后端与其清单规范解耦。

## 可用的后端

| Backend   | Use Case |
|---------|----------|
| [**`pixi-build-cmake`**](./backends/pixi-build-cmake.md) | 使用 CMake 的项目 |
| [**`pixi-build-python`**](./backends/pixi-build-python.md) | 构建 Python 包 |
| [**`pixi-build-rattler-build`**](./backends/pixi-build-rattler-build.md) | 使用 `recipe.yaml` 直接构建，完全控制 |
| [**`pixi-build-ros`**](./backends/pixi-build-ros.md) | ROS（机器人操作系统）包 |
| [**`pixi-build-r`**](./backends/pixi-build-r.md) | 使用 `R CMD INSTALL` 的 R 包 |
| [**`pixi-build-rust`**](./backends/pixi-build-rust.md) | 基于 Cargo 的 Rust 应用程序和库 |
| [**`pixi-build-mojo`**](./backends/pixi-build-mojo.md) | Mojo 应用程序和包 |

所有后端均可通过 [conda-forge](https://prefix.dev/channels/conda-forge) conda 通道（channel）获取，并支持多个平台（Linux、macOS、Windows）。
对于最新版本的后端，您可以在通道列表前添加 [prefix.dev/pixi-build-backends](https://prefix.dev/channels/pixi-build-backends) conda 通道（channel）。

## 关键概念

- [编译器](./key_concepts/compilers.md) - pixi-build 如何与 conda-forge 的编译器基础设施集成

### 安装

通过将构建后端添加到项目清单（manifest）文件的 `package.build` 部分来安装特定的构建后端：

```toml
--8<-- "docs/source_files/pixi_tomls/simple_pixi_build.toml:build-system"
```

对于自定义后端通道（channel），您可以将通道（channel）添加到项目清单（manifest）文件的 `channels` 部分：
```toml
--8<-- "docs/source_files/pixi_tomls/pixi-build-backend-channel.toml:build"
```

### 覆盖构建后端

有时您想覆盖 Pixi 使用的构建后端。意思是覆盖在 [`[package.build]`](../reference/pixi_manifest.md#build-table) 中指定的后端。我们目前有两个环境变量允许这样做：

1. `PIXI_BUILD_BACKEND_OVERRIDE`：此环境变量允许覆盖一个或多个后端。使用 `{name}={path}` 指定映射到路径的后端名称，并使用 `,` 分隔多个后端。
例如：`pixi-build-cmake=/path/to/bin,pixi-build-python` 将：
   1. 用位于 `/path/to/bin` 的可执行文件覆盖 `pixi-build-cmake` 后端
   2. 并将使用 PATH 中的 `pixi-build-python` 后端
2. `PIXI_BUILD_BACKEND_OVERRIDE_ALL`：如果此环境变量设置为*某个*值，例如 `1` 或 `true`，它将不会单独安装任何后端，并假定所有后端都已被覆盖并在 PATH 中可用。这对于开发目的很有用。例如 `PIXI_BUILD_BACKEND_OVERRIDE_ALL=1 pixi install`

## 故障排除

### 重新构建生成的食谱

当您使用 `pixi build` 构建包时，构建后端会生成一个完整的 rattler-build 食谱，存储在项目的构建目录中。这对于调试构建问题或准确了解包的构建方式很有用。

### 食谱位置

构建后端在两个位置生成食谱：

#### 1. 通用食谱（所有输出）

```
<your_project>/.pixi/build/work/<package-name>--<hash>/debug/
```

此目录包含：

- `recipe.yaml` - 可构建所有包输出的通用食谱
- `variants.yaml` - 包的所有变体配置

#### 2. 特定于变体的食谱（单个输出）

```
<your_project>/.pixi/build/work/<package-name>--<hash>/debug/recipe/<variant_hash>/
```

此目录包含：

- `recipe.yaml` - 构建后端生成的完整 rattler-build 食谱
- `variants.yaml` - 用于此特定构建的变体配置

### 重新构建包

要使用相同配置调试或重新构建包，您有两个选择：

#### 选项 1：导航到食谱目录

1. 导航到食谱目录：
   ```bash
   cd .pixi/build/work/<package-name>--<hash>/recipe/<variant_hash>/debug/
   ```

2. 使用 `rattler-build` 重新构建包：
   ```bash
   rattler-build build
   ```

#### 选项 2：指向食谱目录

使用 `--recipe` 标志在不更改目录的情况下构建：

```bash
rattler-build build --recipe .pixi/build/work/<package-name>--<hash>/debug/recipe/<variant_hash>/
```

这允许您：

- 检查生成的准确食谱
* 使用直接访问 `rattler-build` 调试构建失败
* 了解构建后端如何将您的项目模型（`pixi.toml`）转换

!!! tip
    `<variant_hash>` 确保每个唯一的变体组合都有其自己的食谱目录，让您可以轻松比较不同的构建配置。

### 调试 JSON-RPC

您可以在同一目录中找到项目模型和请求/响应的 JSON 版本以及 `recipe.yaml`。
我们存储：

- 项目模型：`project_model.json`
- 请求：`*_params.json`
- 响应：`*_response.json`
