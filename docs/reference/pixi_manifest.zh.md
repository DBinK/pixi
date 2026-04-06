`pixi.toml` 是工作区清单，也称为 Pixi 工作区配置文件。
它指定工作区的环境，以及这些环境的包依赖要求。它还可以指定可以在这些环境中运行的任务，以及许多其他配置选项。

`toml` 文件以不同的表结构组织。本文档将解释不同表的用法。

!!! tip
    我们也支持 `pyproject.toml` 文件。它的结构与 `pixi.toml` 文件相同，只是你需要用 `tool.pixi` 而非仅用表名作为前缀。
    例如，`[workspace]` 表变成 `[tool.pixi.workspace]`。
    `pyproject.toml` 文件中还有一些可用的小技巧，请查看 [pyproject.toml](../python/pyproject_toml.md) 文档了解更多信息。

## 清单发现

清单可以在以下位置找到。

| **优先级** | **位置**                                                           | **注释**                                                                       |
|-----------|-------------------------------------------------------------------|--------------------------------------------------------------------------------|
| 6         | `--manifest-path`                                                 | 命令行参数                                                                      |
| 5         | `pixi.toml`                                                       | 在你当前的工作目录中。                                                           |
| 4         | `pyproject.toml`                                                  | 在你当前的工作目录中。                                                           |
| 3         | `pixi.toml` 或 `pyproject.toml`                                   | 遍历所有父目录。使用发现的第一个清单。                                            |
| 1         | `$PIXI_PROJECT_MANIFEST`                                          | 如果设置了 `$PIXI_IN_SHELL`。这发生在 `pixi shell` 或 `pixi run` 时。            |

!!! note
    如果存在多个位置，将使用具有最高优先级的清单。

## `workspace` 表

`workspace` 表中最少需要的信息是：

```toml
--8<-- "docs/source_files/pixi_tomls/simple_pixi.toml:project"
```

### `channels`

这是用于获取包的通道列表。
如果你想使用托管在 `anaconda.org` 上的通道，你只需要直接使用通道的名称。

```toml
--8<-- "docs/source_files/pixi_tomls/lots_of_channels.toml:project_channels_long"
```

文件系统上的通道也支持，使用**绝对**路径：

```toml
--8<-- "docs/source_files/pixi_tomls/lots_of_channels.toml:project_channels_path"
```

要访问 [prefix.dev](https://prefix.dev/channels) 或 [Quetz](https://github.com/mamba-org/quetz) 上的私有或公共通道，请使用包含主机名的 URL：

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_channels"
```

### `platforms`

定义工作区支持的平台列表。Pixi 为所有这些平台求解依赖并将它们放入 lock file（`pixi.lock`）。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_platforms"
```

可用平台（除了 `noarch` 和 `unknown`）列在[此处](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/platform/enum.Platform.html)。

!!! tip "特殊的 macOS 和 Windows ARM 行为"
    要在 macOS 或 Windows 上同时支持两种架构，请在你的列表中同时包含两个平台。

    ```toml
    # 所有平台的特定环境
    platforms = ["osx-64", "osx-arm64", "win-64", "win-arm64"]
    # 在 Intel 和 ARM 设置上运行但使用模拟器的环境：
    platforms = ["osx-64", "win-64"]
    ```

    | 操作系统   | 平台      | 架构     | 注释                                                                                                                                  |
    |----------|----------|---------|---------------------------------------------------------------------------------------------------------------------------------------|
    | macOS    | `osx-64` | Intel    | 通过 [Rosetta](https://developer.apple.com/documentation/apple-silicon/about-the-rosetta-translation-environment) 在 Apple Silicon 上运行         |
    | macOS    | `osx-arm64` | Apple Silicon | Apple Silicon 的优化二进制文件（推荐）。                                                                                    |
    | Windows  | `win-64` | Intel/AMD64 | 通过 [Prism](https://learn.microsoft.com/en-us/windows/arm/apps-on-arm-x86-emulation) 在 ARM 处理器上运行                         |
    | Windows  | `win-arm64` | ARM64   | ARM 的优化二进制文件，并非所有包都支持。                                                                                                 |


### `name`（可选）

工作区的名称。
如果未指定名称，则使用包含工作区的目录的名称。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_name"
```

### `version`（可选）

工作区的版本。
这应该是基于 conda Version Spec 的有效版本。
有关版本规范中允许的内容的解释，请参阅[版本文档](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/struct.Version.html)。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_version"
```

### `authors`（可选）

工作区作者列表。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_authors"
```

### `description`（可选）

这应该包含工作区的简短描述。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_description"
```

### `license`（可选）

作为有效的 [SPDX](https://spdx.org/licenses/) 字符串的许可证（例如 MIT AND Apache-2.0）

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_license"
```

### `license-file`（可选）

许可证文件的相对路径。

```toml
license-file = "LICENSE.md"
```

### `readme`（可选）

README 文件的相对路径。

```toml
readme = "README.md"
```

### `homepage`（可选）

工作区主页的 URL。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_homepage"
```

### `repository`（可选）

工作区源代码仓库的 URL。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_repository"
```

### `documentation`（可选）

工作区文档的 URL。

```toml
--8<-- "docs/source_files/pixi_tomls/main_pixi.toml:project_documentation"
```

### `conda-pypi-map`（可选）

通道名称或 URL 到映射位置的映射，映射可以是 URL/Path。
映射应以 `json` 格式构建，其中 `conda_name`：`pypi_package_name`。
示例：

```json title="local/robostack_mapping.json"
{
  "jupyter-ros": "my-name-from-mapping",
  "boltons": "boltons-pypi"
}
```

如果 `conda-forge` 不在 `conda-pypi-map` 中，`pixi` 将使用 `prefix.dev` 映射。

```toml
conda-pypi-map = { conda-forge = "https://example.com/mapping", "https://repo.prefix.dev/robostack" = "local/robostack_mapping.json"}
```

也可以通过在列表中添加空映射来禁用外部映射获取

```toml
conda-pypi-map = { conda-forge = "map.json" }
```

```json title="map.json"
{}
```

### `channel-priority`（可选）

这是求解步骤中通道优先级的设置。

选项：

- `strict`：**默认**，通道按其在 `channels` 列表中定义的顺序使用。
  只使用第一个包含该包的通道中的包。
  这确保不会将来自不同通道的单个包的不同变体混合使用。
  使用来自不兼容通道（如 `conda-forge` 和 `main`）的包可能导致难以调试的 ABI 不兼容。

  我们强烈建议不要更改默认值。

- `disabled`：没有优先级，所有通道的所有包变体将按包名称分组并作为一个整体求解。
  使用此选项时应小心。
  当你使用此模式时，包变体可能来自**任何**通道，包可能不兼容。
  这可能导致难以调试的 ABI 不兼容。

  我们强烈反对使用此选项。

```toml
channel-priority = "disabled"
```
!!! warning "`channel-priority = \"disabled\"` 是安全风险"
    禁用通道优先级可能导致不可预测的依赖解析。
    这是一个可能的安全风险，因为它可能导致安装来自意外通道的包。
    建议保持默认的严格设置并仔细考虑通道顺序。
    如有必要，直接为依赖指定通道。
    ```toml
    [workspace]
    # 将 conda-forge 放在第一位可以解决大多数问题
    channels = ["conda-forge", "channel-name"]
    [dependencies]
    package = {version = "*", channel = "channel-name"}
    ```

### `solve-strategy`（可选）

这是求解步骤中使用的策略设置。

选项：

- `highest`：**默认**，将所有包求解到最高兼容版本。

- `lowest`：将所有包求解到最低兼容版本。

- `lowest-direct`：仅将直接依赖包求解到最低兼容版本。传递依赖仍使用 `highest` 策略排序。

```toml
solve-strategy = "lowest"
```

!!! note
    当环境中使用的多个功能设置了特定的求解策略时，
    使用环境中最左边声明的功能的求解策略。
    ```toml
    [feature.one]
    solve-strategy = "lowest"
    [feature.two]
    solve-strategy = "lowest-direct"
    [environments]
    combined = ["two", "one"] # <- 使用功能 `two` 的求解策略
    ```

### `requires-pixi`（可选）

解析和构建工作区所需的 `pixi` 本身版本规范。如果未设置（**默认**），任何版本都可以。如果设置，它必须是有效的 conda 版本规范字符串，并且运行中的 `pixi` 版本必须在解析或构建工作区之前匹配所需规范，否则如果版本不匹配则退出并报错。

例如，使用以下清单，`pixi shell` 将在 `pixi 0.39.0` 上失败，但在升级到 `pixi 0.40.0` 后成功：

```toml
[workspace]
requires-pixi = ">=0.40"
```

上限也可以像这样限制：

```toml
[workspace]
requires-pixi = ">=0.40,<1.0"
```

!!! note
    此选项应用于提高构建工作区的可重现性。复杂的要求规范可能是设置构建环境的障碍。

### `exclude-newer`（可选）

指定后，这将排除任何比指定日期更新的包。这对于无论新包发布如何都可以复现安装很有用。

日期可以按以下格式指定：

* 作为 [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339.html) 时间戳（例如 `2023-10-01T00:00:00Z`）
* 作为 `YYYY-MM-DD` 格式的日期（例如 `2023-10-01`），使用系统时区。

PyPI 和 conda 包都会被考虑。

!!! note
    请注意，对于 Pypi 包索引，包索引必须支持 [`PEP 700`](https://peps.python.org/pep-0700/) 中指定的 `upload-time` 字段。
    如果给定分发不存在该字段，该分发将被视为不可用。PyPI 为所有包提供 `upload-time`。

### `build-variants`（可选）

!!! warning "预览功能"
    构建变体需要启用 `pixi-build` 预览功能：
    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

构建变体允许你为工作区中的包指定不同的依赖版本，创建一个针对多个配置的"构建矩阵"。这对于针对不同编译器版本、Python 版本或其他关键依赖测试包特别有用。

构建变体定义为键值对，其中每个键表示依赖名称，值是要构建的版本规范列表。

#### 基本用法

```toml
[workspace.build-variants]
python = ["3.11.*", "3.12.*"]
c_compiler_version = ["11.4", "14.0"]
```

#### 构建变体如何工作

指定构建变体时，Pixi 将：

1. **创建变体组合**：生成指定变体的所有可能组合
2. **构建单独的包**：为每个变体组合创建不同的包构建
3. **解析依赖**：确保每个变体与兼容的依赖版本解析
4. **生成唯一构建字符串**：每个变体在包名称中获得唯一构建标识符

#### 平台特定变体

构建变体也可以按平台指定：

```toml
[workspace.build-variants]
python = ["3.11.*", "3.12.*"]

# Windows 特定变体
[workspace.target.win-64.build-variants]
python = ["3.11.*"]  # 仅 Windows 上的 Python 3.11
c_compiler = ["vs2019"]

# Linux 特定变体
[workspace.target.linux-64.build-variants]
c_compiler = ["gcc"]
c_compiler_version = ["11.4", "13.0"]
```

#### 常见用例

- **多版本 Python 包**：针对 Python 3.11 和 3.12 构建
- **编译器变体**：针对不同编译器版本测试 C/C++ 包
- **依赖兼容性**：确保包与关键依赖的不同版本配合使用
- **跨平台构建**：不同操作系统有不同的构建配置

有关详细示例和教程，请参阅[构建变体文档](../build/variants.md)。

### `build-variants-files`（可选）

!!! warning "预览功能"
    构建变体文件需要启用 `pixi-build` 预览功能：
    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

使用 `build-variants-files` 从 YAML 文件引用外部变体定义。
路径相对于工作区根目录解析并按列出的顺序处理——早期文件中的条目优先于后期加载的值。

```toml
[workspace]
build-variants-files = [
    "./pinning/conda_build_config.yaml",
    "./variants/overrides.yaml",
]
```

每个条目必须指向 `conda_build_config.yaml` 或定义构建变体的另一个 `.yaml` 文件。
如果文件名为 `conda-build_config.yaml`，它将尝试使用 [`conda-build` 的变体语法](https://docs.conda.io/projects/conda-build/en/stable/resources/variants.html#using-variants-with-the-conda-build-api)的子集来解析它。
否则，它将使用 [rattler-build 文档](https://rattler.build/latest/variants/#variant-configuration)中概述的 rattler-build 语法。

## `tasks` 表

任务是自动化工作区中某些自定义命令的方式。例如，`lint` 或 `format` 步骤。
Pixi 工作区中的任务本质上是跨平台的 shell 命令，跨平台语法统一。
有关更深入的信息，请查看[高级任务文档](../workspace/advanced_tasks.md)。
Pixi 的任务使用 `pixi run` 在 Pixi 环境中运行，并使用 [`deno_task_shell`](../workspace/advanced_tasks.md#our-task-runner-deno_task_shell) 执行。

```toml
[tasks]
simple = "echo This is a simple task"
cmd = { cmd="echo Same as a simple task but now more verbose" }
depending = { cmd="echo run after simple", depends-on="simple" }
alias = { depends-on=["depending"] }
download = { cmd="curl -o file.txt https://example.com/file.txt" , outputs=["file.txt"] }
build = { cmd="npm build", cwd="frontend", inputs=["frontend/package.json", "frontend/*.js"] }
run = { cmd="python run.py $ARGUMENT", env={ ARGUMENT="value" }} # 设置环境变量
backend = { cmd="pytest", env={ BACKEND="{{ backend }}" }, args=[{arg="backend", default="numpy"}] } # 模板字符串中的环境变量
format = { cmd="black $INIT_CWD" } # 在你运行 pixi run format 的地方运行 black
clean-env = { cmd="python isolated.py", clean-env=true } # 仅在 Unix 上！
test = { cmd="pytest", default-environment="test" }  # 设置默认 pixi 环境
```

你可以使用 [`pixi task`](cli/pixi/task.md) 修改此表。
!!! note
    使用 [target](#the-target-table) 表为不同平台指定不同的任务

!!! info
    如果你想在 `pixi task list` 或 `pixi info` 中隐藏任务，你可以在名称前加上下划线。
    例如，如果你想隐藏 `depending`，你可以将其重命名为 `_depending`。

## `system-requirements` 表

系统要求用于定义依赖解析期间使用的最小系统规格。

例如，我们可以定义一个具有特定最小 libc 版本的 unix 系统。
```toml
[system-requirements]
libc = "2.28"
```
或使工作区依赖特定版本的 `cuda`：
```toml
[system-requirements]
cuda = "12"
```

选项是：

- `linux`：Linux 内核的最小版本。
- `libc`：libc 库的最小版本。也允许指定 libc 库的家族。
  例如 `libc = { family="glibc", version="2.28" }`
- `macos`：macOS 操作系统的最小版本。
- `cuda`：CUDA 库的最小版本。

更多信息请参阅[系统要求文档](../workspace/system_requirements.md)。

## `pypi-options` 表

`pypi-options` 表用于定义特定于 PyPI 注册表的选项。
这些选项可以在根级别指定，这将把它添加到默认选项功能，或者在功能级别指定，当功能包含在环境中时，这些选项将合并。

可以定义的选项是：

- `index-url`：替换主索引 URL。
- `extra-index-urls`：添加额外的索引 URL。
- `find-links`：类似于 `pip` 中的 `--find-links` 选项。
- `no-build-isolation`：禁用构建隔离，只能按包设置。
- `no-build`：不构建源码分发。
- `no-binary`：不使用预构建的 wheel。
- `index-strategy`：允许指定要使用的索引策略。
- `prerelease-mode`：控制是否允许预发布版本进行依赖解析。
- `skip-wheel-filename-check`：允许安装文件名和元数据版本不匹配的 wheel。

这些选项在以下部分中解释。这些选项大多直接取自或稍作修改自 [uv 设置](https://docs.astral.sh/uv/reference/settings/)。如果你需要任何缺失的选项，欢迎[提交](https://github.com/prefix-dev/pixi/issues)请求。

### 替代注册表

!!! info "严格索引优先级"
    与 pip 不同，因为我们使用 uv，我们有严格的索引优先级。这意味着第一个能找到包的索引被使用。
    顺序由 toml 文件中的顺序决定。`extra-index-urls` 优先于 `index-url`。在 [uv 文档](https://docs.astral.sh/uv/pip/compatibility/#packages-that-exist-on-multiple-indexes) 上了解更多。

通常，你可能想为你的工作区使用替代或额外索引。这可以通过将 `pypi-options` 表添加到你的 `pixi.toml` 文件来完成，以下选项可用：

- `index-url`：替换主索引 URL。如果未设置，默认使用的索引是 `https://pypi.org/simple`。
  每个环境只能定义**一个** `index-url`。
- `extra-index-urls`：添加额外的索引 URL。URL 按定义顺序使用。并且优先于 `index-url`。这些跨功能合并到环境中。
- `find-links`：可以是路径 `{path = './links'}` 或 URL `{url = 'https://example.com/links'}`。
  这类似于 `pip` 中的 `--find-links` 选项。这些跨功能合并到环境中。

示例：

```toml
[pypi-options]
index-url = "https://pypi.org/simple"
extra-index-urls = ["https://example.com/simple"]
find-links = [{path = './links'}]
```

Pixi 仓库中有一些利用此功能的[示例](https://github.com/prefix-dev/pixi/tree/main/examples/pypi-custom-registry)。

!!! tip "认证方法"
    要了解私有注册表的现有认证方法，请查看 [PyPI 认证](../deployment/authentication.md#pypi-authentication) 部分。

### 无构建隔离

虽然构建隔离是一个好的默认值。
你可以选择**不为**某个包名隔离构建，这允许构建访问 `pixi` 环境。
这在你想为构建过程使用 `torch` 或类似的东西时很方便。

```toml
[dependencies]
pytorch = "2.4.0"

[pypi-options]
no-build-isolation = ["detectron2"]

[pypi-dependencies]
detectron2 = { git = "https://github.com/facebookresearch/detectron2.git", rev = "5b72c27ae39f99db75d43f18fd1312e1ea934e60"}
```

设置 `no-build-isolation` 也会影响 PyPI 包的安装顺序。
包按以下顺序安装：
- conda 包一次性安装
- 带构建隔离的包一次性安装
- 无构建隔离的包按添加到 `no-build-isolation` 的顺序安装

也可以通过将 `no-build-isolation` 设置为 `true` 来从构建隔离中移除所有包。

```toml
[pypi-options]
no-build-isolation = true
```

!!! tip "Conda 依赖定义构建环境"
    要有效地使用 `no-build-isolation`，使用 conda 依赖定义构建环境。
    这些在 PyPI 依赖解析之前安装，这样这些依赖在构建过程中可用。在上面的例子中添加 `torch` 作为 PyPI 依赖将是无效的，因为在 PyPI 解析阶段它还没有安装。

### 无构建

启用后，解析不会运行任意 Python 代码。将重用已构建源码分发的缓存 wheel，但需要构建分发的操作将退出并报错。

可以按包或全局设置。
```toml
[pypi-options]
# 不允许 sdists
no-build = true # 默认是 false
```
或：
```toml
[pypi-options]
no-build = ["package1", "package2"]
```

功能合并时，遵循以下优先级：
`no-build = true` > `no-build = ["package1", "package2"]` > `no-build = false`
所以，展开来说：如果环境中任何功能设置了 `no-build = true`，
这将被用作环境的设置。

### 无二进制
不安装预构建的 wheel。

给定的包将从源码构建和安装。解析器仍将使用预构建的 wheel 来提取包元数据（如果有）。

可以按包或全局设置。

```toml
[pypi-options]
# 从不使用预构建的 wheel
no-binary = true # 默认是 false
```
或：
```toml
[pypi-options]
no-binary = ["package1", "package2"]
```

功能合并时，遵循以下优先级：
`no-binary = true` > `no-binary = ["package1", "package2"]` > `no-binary = false`
所以，展开来说：如果环境中任何功能设置了 `no-binary = true`，
这将被用作环境的设置。

### 索引策略

解析多个索引 URL 时使用的策略。描述修改自 [uv](https://docs.astral.sh/uv/reference/settings/#index-strategy) 文档：

默认情况下，`uv` 和 `pixi` 会在给定包可用的第一个索引处停止，并将解析限制在第一个索引上存在的那些（first-match）。这可以防止**依赖混淆**攻击，攻击者可以将恶意包上传到次要索引使用相同的名称。

!!! warning "每个环境一个索引策略"
    每个环境或 solve-group 只能定义一个 `index-strategy`，否则将显示错误。

可能的值：

- **"first-index"**：仅使用第一个返回给定包名称匹配的索引的结果
- **"unsafe-first-match"**：在所有索引中搜索每个包名称，耗尽第一个索引的版本后再继续下一个。意味着如果包 `a` 在索引 `x` 和 `y` 上可用，它将首选来自 `x` 的版本，除非你请求的包版本**仅**在 `y` 上可用。
- **"unsafe-best-match"**：在所有索引中搜索每个包名称，首选找到的**最佳**版本。如果包版本在多个索引中，仅查看第一个索引的条目。所以给定包含包 `a` 的索引 `x` 和 `y`，它将从 `x` 或 `y` 中获取**最佳**版本，但如果**该版本**在两个索引上都可用，它将首选 `x`。

!!! info "仅 PyPI"
    `index-strategy` 仅更改 PyPI 包解析，不更改 conda 包解析。

### 预发布模式

在依赖解析期间考虑预发布版本时使用的策略。描述取自 [uv 文档](https://docs.astral.sh/uv/reference/settings/#prerelease)。

默认情况下，当包只有预发布版本可用时，或在版本说明符中明确请求预发布版本时（例如 `>=1.0.0a1`），`pixi` 将允许预发布版本。

!!! warning "每个环境一个预发布模式"
    每个环境或 solve-group 只能定义一个 `prerelease-mode`，否则将显示错误。

可能的值：

- **"disallow"**：禁止所有预发布版本。
- **"allow"**：允许所有预发布版本。
- **"if-necessary"**：如果包的所有版本都是预发布版本，则允许预发布版本。
- **"explicit"**：允许在版本要求中有明确预发布标记的第一方包的预发布版本。
- **"if-necessary-or-explicit"**（默认）：如果包的所有版本都是预发布版本，或者包在版本要求中有明确的预发布标记，则允许预发布版本。

示例：
```toml
[pypi-options]
prerelease-mode = "allow"  # 允许所有预发布版本
```

### 跳过 Wheel 文件名检查

默认情况下，`uv` 验证 wheel 文件名与 wheel 内的包元数据（名称和版本）匹配。此验证确保 wheel 命名正确，并有助于防止安装格式错误的包。

但是，在某些情况下，你可能需要安装文件名版本与元数据版本不匹配的 wheel。`skip-wheel-filename-check` 选项允许你禁用此验证。

!!! warning "每个环境一个 skip-wheel-filename-check"
    每个环境或 solve-group 只能定义一个 `skip-wheel-filename-check`，否则将显示错误。

可能的值：

- **`false`**（默认）：执行 wheel 文件名验证。如果文件名和元数据不匹配，安装将失败。
- **`true`**：跳过 wheel 文件名验证。允许安装文件名和元数据版本不匹配的 wheel。

#### 优先级

`UV_SKIP_WHEEL_FILENAME_CHECK` 环境变量优先于 `skip-wheel-filename-check` pypi-option。这允许临时覆盖而无需修改清单。

示例：
```toml
[pypi-options]
# 允许安装格式错误的 wheel
skip-wheel-filename-check = true 
```

或按功能设置：
```toml
[feature.special.pypi-options]
# 仅用于此功能的环境
skip-wheel-filename-check = true
```

## `dependencies` 表（们）
??? info "依赖项的详细信息"
    有关依赖项类型的更多详细信息，请务必查看[运行、主机、构建](../build/dependency_types.md) 依赖项文档。

本节定义了你希望用于工作区的依赖项。

有多个依赖项表。
默认是 `[dependencies]`，这些是跨平台共享的依赖项。

依赖项使用 [VersionSpec](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/version_spec/enum.VersionSpec.html) 定义。
`VersionSpec` 结合了 [Version](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/struct.Version.html) 和可选操作符。

!!! tip "需要指定构建字符串或更具体的包？"
    有关指定构建字符串和高级包规格的详细信息，请参阅[包规格](../concepts/package_specifications.md) 指南。

一些示例：

```toml
# 使用这个精确的包版本
package0 = "==1.2.3"
# 使用 1.2.3 到 1.3.0
package1 = "~=1.2.3"
# 大于 1.2 小于等于 1.4
package2 = ">1.2,<=1.4"
# 大于等于 1.2.3 或小于 1.0.0（不包括 1.0.0）
package3 = ">=1.2.3|<1.0.0"
```

依赖项也可以定义为映射，使用 matchspec

```toml
package0 = { version = ">=1.2.3", channel="conda-forge" }
package1 = { version = ">=1.2.3", build="py34_0" }
```

!!! tip
    依赖项可以使用 `pixi add` 命令行轻松添加。
    运行 `add` 对现有依赖项将替换为它能使用的最新版本。

!!! note
    要为不同平台指定不同的依赖项，请使用 [target](#the-target-table) 表

### `dependencies`

添加你希望安装到环境中的任何 conda 包依赖项。
如果使用 `conda-forge` 以外的任何内容，请记得将通道添加到 `workspace` 表。
即使依赖项定义了通道，该通道也应该添加到 `workspace.channels` 列表中。

```toml
[dependencies]
python = ">3.9,<=3.11"
rust = "==1.72"
pytorch-cpu = { version = "~=1.1", channel = "pytorch" }
```

### `constraints`

`constraints` 表让你限制可以安装的 conda 包版本，而不需要显式要求它们。
仅当包实际上存在于环境中时（被另一个依赖项引入）才强制执行约束；如果不需要该包，约束将被静默忽略。

这在两种常见场景中很有用：

- **阻止已知不良版本**——你已知某个传递依赖在某个版本范围内存在回归，你想在不加硬依赖的情况下将其排除。
- **协调可选包**——你的项目有或没有可选库都可以工作，但如果安装了该库，它必须是特定一代（例如 CUDA 12 vs 11）。

```toml
[workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "win-64"]

[dependencies]
# openssl 将被传递引入；这个约束
# 确保我们永远不会得到有漏洞的 1.x 版本。
requests = ">=2.28"

[constraints]
openssl = ">=3.0"
```

约束使用与 `[dependencies]` 相同的 [VersionSpec](https://docs.rs/rattler_conda_types/latest/rattler_conda_types/version_spec/enum.VersionSpec.html) 语法。

!!! note
    约束**不会**导致安装包。它们只是限制*如果*包已经被安装则可接受的版本。当你实际需要包存在时使用 `[dependencies]`。

#### 平台特定约束

与 `[dependencies]` 一样，约束可以使用 [`[target]`](#the-target-table) 表设为平台特定：

```toml
[constraints]
openssl = ">=3.0"

[target.linux-64.constraints]
# 在 Linux 上进一步收紧约束
openssl = ">=3.0.7"
```

#### 每个功能的约束和合并

约束也可以限定到 [feature](#the-feature-table)。
当环境由多个功能组成时，**所有活动功能**的约束被**连接**起来，就像 `[dependencies]` 一样。
这意味着每个功能可以独立约束传递依赖，并且生成的环境必须同时满足所有这些约束。

```toml
[dependencies]
python = ">=3.11"

[feature.cuda.dependencies]
pytorch-gpu = ">=2.0"

# 当 cuda 功能激活时，强制兼容的 CUDA 工具包版本
[feature.cuda.constraints]
cuda = ">=12.0"

[feature.cuda11.constraints]
cuda = "<12"

[environments]
gpu = ["cuda"]
legacy-gpu = ["cuda11"]
```

在 `gpu` 环境中，求解器看到 `cuda = ">=12.0"` 作为约束；
在 `legacy-gpu` 环境中，它看到 `cuda = "<12"`。
如果两个功能在同一环境中都是活动的，求解器将收到两个约束，并且需要找到一个满足所有这些约束的版本。

### `pypi-dependencies`

??? info "PyPI 集成的详细信息"
    我们使用 [`uv`](https://github.com/astral-sh/uv)，这是一个用 Rust 编写的快速 pip 替代品。

    我们将 uv 作为库集成，因此我们使用 uv 求解器，将 conda 包作为"锁定"传递。
    这不允许 uv 自己安装这些依赖，并确保它使用这些包的确切版本进行解析。
    这在基于 conda 的包管理器中是独特的，通常只是从子进程中调用 pip。

    uv 解析直接包含在 lockfile 中。

Pixi 直接支持依赖 PyPI 包，PyPA 称分布式包为"distribution"。
有[源码分发](https://packaging.python.org/en/latest/specifications/source-distribution-format/)和[二进制分发](https://packaging.python.org/en/latest/specifications/binary-distribution-format/)，两者都受 pixi 支持。
这些分布在 conda 环境解析和安装后安装到环境中。
PyPI 包未在 [prefix.dev](https://prefix.dev/channels) 上编制索引，但可以在 [pypi.org](https://pypi.org/) 上查看。

!!! warning "重要注意事项"
    - **稳定性**：PyPI 包可能不如它们的 conda 对应物稳定。尽可能优先在 `dependencies` 表中使用 conda 包。

#### 版本规格：

这些依赖项不遵循 [conda MatchSpec](https://rattler.prefix.dev/py-rattler/match_spec#matchspec) 规格。
`version` 是根据 [PEP404/PyPA](https://packaging.python.org/en/latest/specifications/version-specifiers/) 的版本字符串规格。
此外，可以包含 extra 列表，它们本质上是可选依赖。
请注意，此 `version` 与 conda MatchSpec 类型不同。
请参阅下面的示例以查看它在实践中是如何使用的：

```toml
[dependencies]
# 使用 pypi-dependencies 时，需要 python 来解析 pypi 依赖项
# 确保包含这个
python = ">=3.6"

[pypi-dependencies]
fastapi = "*"  # 任何版本（通配符 `*` 是 pixi 的补充，不属于规格）
pre-commit = "~=3.5.0" # 这是单个版本说明符
# 使用 toml 映射允许用户添加 `extras`
pandas = { version = ">=1.0.0", extras = ["dataframe", "sql"]}

# git 依赖项
# 使用 ssh
flask = { git = "ssh://git@github.com/pallets/flask" }
# 使用 https 和特定修订版
httpx = { git = "https://github.com/encode/httpx.git", rev = "c7c13f18a5af4c64c649881b2fe8dbd72a519c32"}

# 使用 https 和特定分支
boltons = { git = "https://github.com/mahmoud/boltons.git", branch = "master" }

# 使用 https 和特定标签
boltons = { git = "https://github.com/mahmoud/boltons.git", tag = "25.0.0" }

# 使用 https、特定标签和一些子目录
boltons = { git = "https://github.com/mahmoud/boltons.git", tag = "25.0.0", subdirectory = "some-subdir" }

# 你也可以直接从路径添加源码依赖项，保持相对于工作区根目录
minimal-project = { path = "./minimal-project", editable = true}

# 你也可以直接使用 url，可以是 `.tar.gz` 或 `.zip`，或 `.whl` 文件
click = { url = "https://github.com/pallets/click/releases/download/8.1.7/click-8.1.7-py3-none-any.whl" }

# 你也可以只使用默认 git 仓库，它将检出默认分支
pytest = { git = "https://github.com/pytest-dev/pytest.git"}
```

!!! warning "使用 git SSH URL"
    在 git 依赖项中使用 SSH URL 时，请确保将你的 SSH 密钥添加到你的 SSH agent。
    你可以通过运行 `ssh-add` 来完成，这会提示你输入 SSH 密钥 passphrase。确保 `ssh-add` agent 或服务正在运行，并且你已生成公钥/私钥。要了解更多详细信息，请查看 [Github SSH 文档](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)。

#### 完整规格

Pixi 支持的 PyPI 依赖项的完整规格可以分为以下字段：

##### `extras`

要与包一起安装的 extras 列表。例如 `["dataframe", "sql"]`
extras 字段与所有其他版本说明符配合使用，因为它是版本说明符的附加部分。

```toml
pandas = { version = ">=1.0.0", extras = ["dataframe", "sql"]}
pytest = { git = "URL", extras = ["dev"]}
black = { url = "URL", extras = ["cli"]}
minimal-project = { path = "./minimal-project", editable = true, extras = ["dev"]}
```

##### `version`

要安装的包的版本。例如 `">=1.0.0"` 或 `*` 表示任何版本，这是 Pixi 特有的。
Version 是我们的默认字段，所以不使用内联表（`{}`）将默认为此字段。

```toml
py-rattler = "*"
ruff = "~=1.0.0"
pytest = {version = "*", extras = ["dev"]}
```

##### `index`

index 参数允许你指定从其安装特定包的自定义包索引 URL。
当你希望确保从特定来源而不是从默认索引获取包时，此功能很有用。

例如，要使用官方 Python 包索引 (PyPI) 以外的某个索引（https://pypi.org/simple），你可以使用 `index` 参数：

```toml
torch = { version = "*", index = "https://download.pytorch.org/whl/cu118" }
```

这在 PyTorch 中特别有用，因为注册表被绑定到不同的 CUDA 版本。
在[这里](../python/pytorch.md)了解更多关于安装 PyTorch 的信息。

##### `git`

要安装的 git 仓库。
同时支持 https:// 和 ssh:// URLs。

将 `git` 与 `rev` 或 `subdirectory` 结合使用：

- `rev`：要安装的特定修订版。例如 `rev = "0106aced5faa299e6ede89d1230bd6784f2c3660"`
- `subdirectory`：要安装的子目录。`subdirectory = "src"` 或 `subdirectory = "src/packagex"`

```toml
# 注意不要忘记 `ssh://` 或 `https://` 前缀！
pytest = { git = "https://github.com/pytest-dev/pytest.git"}
httpx = { git = "https://github.com/encode/httpx.git", rev = "c7c13f18a5af4c64c649881b2fe8dbd72a519c32"}
py-rattler = { git = "ssh://git@github.com/conda/rattler.git", subdirectory = "py-rattler" }
```

##### `path`

要安装的本地路径。例如 `path = "./path/to/package"`
我们建议将你的路径项目保存在工作区中，并使用相对路径。

将 `editable` 设置为 `true` 以可编辑模式安装，这 highly recommended 因为如果你不使用可编辑模式很难重新安装。例如 `editable = true`

```toml
minimal-project = { path = "./minimal-project", editable = true}
```

##### `url`

直接从 URL 安装 wheel 或 sdist 的 URL。

```toml
pandas = {url = "https://files.pythonhosted.org/packages/3d/59/2afa81b9fb300c90531803c0fd43ff4548074fa3e8d0f747ef63b3b5e77a/pandas-2.2.1.tar.gz"}
```

??? tip "你知道你可以使用：`add --pypi` 吗？"
    使用 `add` 命令的 `--pypi` 标志从 CLI 快速添加 PyPI 包。
    例如 `pixi add --pypi flask`

    _这尚不支持 `pypi-dependencies` 表的所有功能。_

#### 源码分发（`sdist`）

[源码分发格式](https://packaging.python.org/en/latest/specifications/source-distribution-format/)是基于源码的格式（简称 sdist），包可以将其与二进制 wheel 格式一起包含。
由于这些分布需要构建，它们需要一个 python 可执行文件来执行此操作。
这就是为什么 python 需要存在于 conda 环境中。
Sdists 通常依赖于系统包来构建，特别是编译 C/C++ 基于 python 绑定时。
例如，考虑依赖 C 库的 Python SDL2 绑定：SDL2。
为了帮助构建这些依赖，我们在解析之前激活包含这些 pypi 依赖项的 conda 环境。
这样，当源码分发依赖例如 `gcc` 时，它会从 conda 环境而不是系统环境中使用。

## `activation` 表

activation 表用于在环境激活时需要运行的专门激活操作。

用户可以在清单中修改两种类型的激活操作：

- `scripts`：环境激活时运行的脚本列表。
- `env`：环境激活时设置的环境变量映射。

这些激活操作将在 `pixi run` 和 `pixi shell` 命令之前运行。

!!! note
    `scripts` 部分中指定的脚本不会直接在 `pixi shell` 中被 source，而是被调用，
    然后它们设置的环境变量在 `pixi shell` 中被设置，因此定义的函数或其他非环境变量
    对环境的修改将被忽略。

!!! note
    激活操作由系统 shell 解释器运行，因为它们在环境可用之前运行。
    这意味着它在 windows 上作为 `cmd.exe` 运行，在 linux 和 osx（Unix）上作为 `bash` 运行。
    仅支持 `.sh`、`.bash` 和 `.bat` 文件。

    并且环境变量在运行激活脚本的 shell 中设置，因此在使用时注意例如 `$` 或 `%`。

    如果你有按平台的脚本或环境变量，请使用 [target](#the-target-table) 表。

```toml
[activation]
scripts = ["env_setup.sh"]
env = { ENV_VAR = "value" }

# 为支持 windows 平台添加以下内容
[target.win-64.activation]
scripts = ["env_setup.bat"]

[target.linux-64.activation.env]
ENV_VAR = "linux-value"

# 你也可以引用现有的环境变量，但这必须为类 Unix 操作系统和 Windows 分开做
[target.unix.activation.env]
ENV_VAR = "$OTHER_ENV_VAR/unix-value"

[target.win.activation.env]
ENV_VAR = "%OTHER_ENV_VAR%\\windows-value"
```

## `target` 表

target 表是允许按平台进行配置的表。
允许你为每个平台设置不同的任务集或依赖项。

target 表目前为以下子表实现：

- [`activation`](#the-activation-table)
- [`dependencies`](#dependencies)
- [`constraints`](#constraints)
- [`tasks`](#the-tasks-table)

target 表使用 `[target.PLATFORM.SUB-TABLE]` 定义。
例如 `[target.linux-64.dependencies]`

平台可以是：

- `win`、`osx`、`linux` 或 `unix`（`unix` 匹配 `linux` 和 `osx`）
- 或任何（更）具体的[目标平台](#platforms)，例如 `linux-64`、`osx-arm64`

子表可以是上面指定的任何一种。

为了更清楚，让我们看下面的例子。
目前，Pixi 将顶层表（如 `dependencies`）与目标特定的表合并为一个集合。
在依赖项的情况下，可以添加或覆盖依赖项。
在下面的示例中，我们对所有目标使用 `cmake`，但在 `osx-64` 或 `osx-arm64` 上将选择不同版本的 python。

```toml
[dependencies]
cmake = "3.26.4"
python = "3.10"

[target.osx.dependencies]
python = "3.11"
```

以下是更多示例：

```toml
[target.win-64.activation]
scripts = ["setup.bat"]

[target.win-64.dependencies]
msmpi = "~=10.1.1"

[target.win-64.build-dependencies]
vs2022_win-64 = "19.36.32532"

[target.win-64.tasks]
tmp = "echo $TEMP"

[target.osx-64.dependencies]
clang = ">=16.0.6"
```

## `feature` 和 `environments` 表

`feature` 表允许你定义可用于创建不同 `[environments]` 的功能。
`[environments]` 表允许你定义不同的环境。设计在[本设计文档](../workspace/multi_environment.md)中解释。

```toml title="Simplest example"
[feature.test.dependencies]
pytest = "*"

[environments]
test = ["test"]
```

这将创建一个名为 `test` 的环境，其中安装了 `pytest`。

### The `feature` 表

`feature` 表允许你为每个功能定义以下字段。

- `dependencies`：与 [dependencies](#dependencies) 相同。
- `constraints`：与 [constraints](#constraints) 相同。
- `pypi-dependencies`：与 [pypi-dependencies](#pypi-dependencies) 相同。
- `pypi-options`：与 [pypi-options](#the-pypi-options-table) 相同。
- `system-requirements`：与 [system-requirements](#the-system-requirements-table) 相同。
- `activation`：与 [activation](#the-activation-table) 相同。
- `platforms`：与 [platforms](#platforms) 相同。除非被覆盖，否则功能的 `platforms` 将是工作区级别定义的。
- `channels`：与 [channels](#channels) 相同。除非被覆盖，否则功能的 `channels` 将是工作区级别定义的。
- `channel-priority`：与 [channel-priority](#channel-priority-optional) 相同。
- `solve-strategy`：与 [solve-strategy](#solve-strategy-optional) 相同。
- `target`：与 [target](#the-target-table) 相同。
- `tasks`：与 [tasks](#the-tasks-table) 相同。

这些表也可以不带 `feature` 前缀使用。
当使用这些时，我们称它们为 `default` feature。这是一个受保护的名字，你不能用于你自己的 feature。

```toml title="Cuda feature table example"
[feature.cuda]
activation = {scripts = ["cuda_activation.sh"]}
# 结果：默认是 `conda-forge` 时为 ["nvidia", "conda-forge"]
channels = ["nvidia"]
dependencies = {cuda = "x.y.z", cudnn = "12.0"}
constraints = {cuda = ">=12.0"}
pypi-dependencies = {torch = "==1.9.0"}
platforms = ["linux-64", "osx-arm64"]
system-requirements = {cuda = "12"}
tasks = { warmup = "python warmup.py" }
target.osx-arm64 = {dependencies = {mlx = "x.y.z"}}
```

```toml title="Cuda feature table example but written as separate tables"
[feature.cuda.activation]
scripts = ["cuda_activation.sh"]

[feature.cuda.dependencies]
cuda = "x.y.z"
cudnn = "12.0"

[feature.cuda.pypi-dependencies]
torch = "==1.9.0"

[feature.cuda.system-requirements]
cuda = "12"

[feature.cuda.tasks]
warmup = "python warmup.py"

[feature.cuda.target.osx-arm64.dependencies]
mlx = "x.y.z"

# Channels and Platforms are not available as separate tables as they are implemented as lists
[feature.cuda]
channels = ["nvidia"]
platforms = ["linux-64", "osx-arm64"]
```

### The `environments` 表

`[environments]` 表允许你使用 `[feature]` 表中定义的功能来定义创建的环境。

环境表使用以下字段定义：

- `features`：包含在环境中的功能。除非将 `no-default-feature` 设置为 `true`，否则默认功能会隐式包含在环境中。
- `solve-group`：solve group 用于在求解阶段将环境组合在一起。
  这对于需要具有相同依赖项但可能扩展它们的环境很有用。
  例如，使用附加测试依赖项测试生产环境。
  这些依赖项将在具有相同 solve group 的所有环境中使用相同的版本。
  但不同的环境包含 solve-groups 依赖项的不同子集。
- `no-default-feature`：是否在该环境中包含默认功能。默认为 `false`，以包含默认功能。

```toml title="Full environments table specification"
[environments]
test = {features = ["test"], solve-group = "test"}
prod = {features = ["prod"], solve-group = "test"}
lint = {features = ["lint"], no-default-feature = true}
```
如上面的示例所示，在最简单的情况下，可以通过仅列出其功能来定义环境：

```toml title="Simplest example"
[environments]
test = ["test"]
```

等于

```toml title="Simplest example expanded"
[environments]
test = {features = ["test"]}
```

当环境包含多个功能（包括默认功能）时：

- 环境的 `activation` 和 `tasks` 是其所有功能的 `activation` 和 `tasks` 的并集。
- 环境的 `dependencies` 和 `pypi-dependencies` 是其所有功能的 `dependencies` 和 `pypi-dependencies` 的并集。
  这意味着如果多个功能对同一包定义要求，两个要求将被合并。
  注意在同一环境中添加的功能之间的冲突要求。
- 环境的 `system-requirements` 是其所有功能的 `system-requirements` 的并集。
  如果多个功能指定对同一系统包的要求，则选择最高版本。
- 环境的 `channels` 是其所有功能的 `channels` 的并集。
  可以在每个功能中指定通道优先级，以确保以正确的顺序在环境中考虑通道。
- 环境的 `platforms` 是其所有功能的 `platforms` 的交集。
  请注意，功能支持的平台（包括默认功能）将被视为工作区级别定义的 `platforms`（除非在功能中覆盖）。
  这意味着通常最好将工作区 `platforms` 设置为跨其所有环境可以支持的所有平台。

## 全局配置

全局配置选项在[全局配置](../reference/pixi_configuration.md) 部分中记录。

## 预览功能

Pixi 有时会引入尚不稳定但我们希望用户测试的新功能。这些功能称为预览功能。默认情况下禁用预览功能，可通过在 workspace manifest 中设置 `preview` 字段来启用。preview 字段是一个字符串数组，用于指定要启用的预览功能，或布尔值 `true` 以启用所有预览功能。

manifest 中预览功能的示例：

```toml
--8<-- "docs/source_files/pixi_tomls/simple_pixi_build.toml:preview"
```

文档中的预览功能将在相关页面上标记为此。

## `dev` 表
`dev` 表允许你依赖源码包的开发依赖项。

```toml
[dev]
my-package = { path = "src/my-package" }
```
这将安装位于 `src/my-package` 的包中定义的 `build-dependencies`、`host-dependencies` 和 `run-dependencies`。
更多信息可以在 [Dev 包](../build/dev.md) 文档中找到。

## `package` 部分
