# Rattler-Build 后端

`pixi-build-rattler-build` 后端使用 rattler-build 食谱构建 conda 包。
此后端专为已有 `recipe.yaml` 文件或需要无法通过当前可用后端实现的定制的项目而设计。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    这就是为什么我们要求用户通过将 "pixi-build" 添加到 `workspace.preview` 来选择加入该功能。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## 概述

rattler-build 后端：

- 使用现有的 `recipe.yaml` 文件作为构建清单
- 支持所有标准 rattler-build 食谱功能和选择器
- 自动处理依赖项解析和虚拟包检测
- 可以从单个食谱构建多个输出

## 用法

要在您的 `pixi.toml` 中使用 rattler-build 后端，请在构建系统配置中指定它：

```toml
[package]
name = "rattler_build_package"
version = "0.1.0"

[package.build]
backend = { name = "pixi-build-rattler-build", version = "*" }
channels = ["https://prefix.dev/conda-forge"]
```

后端期望在以下位置之一找到 rattler-build 食谱文件（按顺序搜索）：

1. `package.build.config.recipe` 如果给定
2. 与包清单相同目录中的 `recipe.yaml` 或 `recipe.yml`
3. 包清单子目录中的 `recipe/recipe.yaml` 或 `recipe/recipe.yml`

如果包与工作区（workspace）定义在同一位置，强烈建议将食谱文件放在自己的 `recipe` 目录中。
在[高级概述](https://rattler.build/latest/highlevel)中了解更多关于 `rattler-build` 及其食谱格式的信息。

!!! warning
    如果您期望您的构建脚本与增量编译兼容
    （重用以前构建的文件以加速未来构建），
    您必须确保这些文件的构建目录设置在根目录之外，以启用增量编译。
    这是因为我们为每个构建使用干净的根目录，
    以确保与做出该假设的食谱兼容。

    在实践中，这可能看起来像在开始构建之前将目录更改为 `../build_dir`，
    或者将 `../build_dir` 作为参数传递给您的构建系统。

## 指定依赖项

我们只允许在项目清单（manifest）中使用源代码依赖项（工作区包）。
当使用 `pixi-build-rattler-build` 时，二进制依赖项不允许出现在项目清单（manifest）中。
这是故意的，因为：

1. rattler-build 食谱是二进制依赖项的真实来源。它已经指定了确切的版本、构建变体以及依赖项是进入 build/host/run
2. 在两处都允许二进制依赖项会造成重复和潜在冲突（例如，食谱说 "python >=3.10" 但项目模型说 "python >=3.9"）
3. 源代码依赖项不同——它们代表从本地源代码构建的工作区包。食谱可以按名称引用它们，但无法知道它们的工作区路径。项目模型提供了这种映射

这样，食谱保持对二进制依赖项的完全控制，而项目模型仅提供食谱无法知道的工作区结构信息。

要指定源代码依赖项，请将它们添加到包清单（manifest）中的 `build-dependencies`、`host-dependencies` 或 `run-dependencies`：

```toml title="pixi.toml"
[package.build-dependencies]
a = { path = "../a" }
```

## 配置选项

rattler-build 后端支持以下 TOML 配置选项：

### `experimental`

- **类型**: `Boolean`
- **默认值**: `false`
- **目标合并行为**: 不允许 - 必须在根级别设置

启用 rattler-build 中的实验性功能。这对于某些高级功能（如多输出食谱的 `cache:` 功能）是必需的。

```toml
[package.build.config]
experimental = true
```

注意：此选项不能设置在特定于目标的配置中。它只能在根 `[package.build.config]` 级别设置。

### `recipe`

- **类型**: `String`（路径）
- **默认值**: 按顺序检查 `recipe.yaml`、`recipe.yml`、`recipe/recipe.yaml`、`recipe/recipe.yml`
- **目标合并行为**: 不允许 - 必须在根级别设置

食谱 YAML 文件的路径。

```toml
[package.build.config]
recipe = "../template/recipe.yaml"
```

注意：此选项不能设置在特定于目标的配置中。它只能在根 `[package.build.config]` 级别设置。

### `debug-dir`

后端始终将 JSON-RPC 请求/响应日志和生成的中间食谱写入工作目录内的 `debug` 子目录（例如 `<work_directory>/debug`）。已弃用的 `debug-dir` 配置选项将被忽略；如果它仍在清单中设置，后端会发出警告以使更改明确。

### `extra-input-globs`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定 glob 完全替换基础 glob

作为构建过程的输入文件包含的额外 glob 模式。这些模式被添加到从食谱源和包目录结构确定的默认输入 glob 中。

```toml
[package.build.config]
extra-input-globs = [
    "patches/**/*",
    "scripts/*.sh",
    "*.md"
]
```

对于特定于目标的配置，平台特定的 glob 完全替换基础：

```toml
[package.build.config]
extra-input-globs = ["*.yaml", "*.md"]

[package.build.target.linux-64.config]
extra-input-globs = ["*.yaml", "*.md", "*.sh", "patches-linux/**/*"]
# linux-64 的结果: ["*.yaml", "*.md", "*.sh", "patches-linux/**/*"]
```

## 构建流程

rattler-build 后端遵循以下构建流程：

1. **食谱发现**：在标准位置定位 `recipe.yaml` 文件
2. **依赖项解析**：从 conda 通道（channel）和工作区（workspace）解析 build、host 和 run 依赖项
3. **虚拟包检测**：自动检测系统虚拟包
4. **构建执行**：运行食谱中指定的构建脚本
5. **包创建**：根据食谱规范创建 conda 包

## 作为环境变量的自定义构建变体

使用 `[workspace.build-variants]` 时，任何不是已识别语言键（如 `python`、`numpy`、`r` 等）的变体键都会在构建期间自动导出为环境变量。

这对于将配置传递给构建脚本而不修改食谱很有用。
例如，要覆盖编译期间使用的 macOS sysroot：

```toml title="pixi.toml"
[workspace.target.osx.build-variants]
CONDA_BUILD_SYSROOT = ["/Library/Developer/CommandLineTools/SDKs/MacOSX15.4.sdk"]
```

在构建期间，`CONDA_BUILD_SYSROOT` 将被设置为构建脚本可用的环境变量。
自定义变体键也可以通过 Jinja 在食谱模板中使用：

```yaml title="recipe.yaml"
build:
  script:
    env:
      MY_FLAG: ${{ my_custom_flag }}
```

```toml title="pixi.toml"
[workspace.build-variants]
my_custom_flag = ["enabled"]
```

## 限制

- 需要现有的 rattler-build 食谱文件 - 无法自动推断构建说明
- 构建配置主要通过食谱文件而不是 `pixi.toml` 控制
- 无法在清单（manifest）中指定二进制依赖项
