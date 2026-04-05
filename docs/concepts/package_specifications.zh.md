# 包规格

将包添加到 Pixi workspace 或全局环境时，你可以使用各种规格来精确控制你想要的包版本和构建。
当包有多个针对不同硬件配置的构建（如 CPU vs GPU）时，这尤其重要。
对于 conda 包，Pixi 使用 [MatchSpec](https://rattler.prefix.dev/py-rattler/match_spec#matchspec) 格式来指定包要求。
对于 PyPI 包，Pixi 使用标准的 [PEP440 版本说明符](https://peps.python.org/pep-0440/)。

## 快速示例

=== "添加"
    ```bash
    --8<-- "docs/source_files/shell/package_specifications.sh:quick-add-examples"
    ```
=== "全局"
    ```bash
    --8<-- "docs/source_files/shell/package_specifications.sh:quick-global-examples"
    ```
=== "执行"
    ```bash
    --8<-- "docs/source_files/shell/package_specifications.sh:quick-exec-examples"
    ```

## 基本版本规格

Pixi 使用 [**conda MatchSpec**](https://rattler.prefix.dev/py-rattler/match_spec#matchspec) 格式来指定包要求。
MatchSpec 允许你精确定义你想要的包版本、构建和 channel。

指定包的最简单方式是按名称和[可选版本操作符](#版本操作符)：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/package_specifications.toml:basic-version"
```

## 完整 MatchSpec 语法

除了简单的版本规格，你还可以使用完整的 [MatchSpec](https://rattler.prefix.dev/py-rattler/match_spec#matchspec) 语法来精确控制你想要的包变体。

### 命令行语法

Pixi 在命令行上支持两种语法：

**1. 等号语法（紧凑）：**
```shell
# Format: package=version=build
pixi add "pytorch=2.0.*=cuda*"
# Only build string (any version)
pixi add "numpy=*=py311*"
```

**2. 方括号语法（显式）：**
```shell
# Format: package [key='value', ...]
pixi add "pytorch [version='2.0.*', build='cuda*']"
# Multiple constraints
pixi add "numpy [version='>=1.21', build='py311*', channel='conda-forge']"
# Build number constraint
pixi add "python [version='3.11.0', build_number='>=1']"
```

两种语法是等价的，可以互换使用。

### TOML 映射语法

在你的 `pixi.toml` 中，使用映射语法以获得完全控制：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/package_specifications.toml:mapping-syntax-full"
```

此语法允许你指定：

- **version**：使用[操作符](#版本操作符)的版本约束
- **build**：构建字符串模式（参见[构建字符串](#构建字符串)）
- **build-number**：构建编号约束（例如 `">=1"`, `"0"`）（参见[构建编号](#构建编号)）
- **channel**：特定 channel 名称或完整 URL（参见 [channel](#channel)）
- **sha256/md5**：用于验证的包校验和（参见[校验和](#校验和-sha256md5)）
- **license**：预期许可证类型（参见[许可证](#许可证)）
- **file-name**：特定包文件名（参见[文件名](#文件名)）

### 版本操作符

Pixi 支持各种版本操作符：

| 操作符 | 含义 | 示例 |
|----------|---------|---------|
| `==`     | 完全匹配 | `==3.11.0` |
| `!=`     | 不等于 | `!=3.8` |
| `<`      | 小于 | `<3.12` |
| `<=`     | 小于或等于 | `<=3.11` |
| `>`      | 大于 | `>3.9` |
| `>=`     | 大于或等于 | `>=3.9` |
| `~=`     | 兼容发布 | `~=3.11.0` (>= 3.11.0, < 3.12.0) |
| `*`      | 通配符 | `3.11.*` (any 3.11.x) |
| `,`      | AND | `">=3.9,<3.12"` |
| `|`     | OR | `"3.10|3.11"` |

### 构建字符串

构建字符串标识同一包版本的特定构建。
它们对于有不同构建的包尤其重要：

- **硬件加速**：CPU、GPU/CUDA 构建
- **Python 版本**：为不同 Python 解释器构建的包
- **编译器变体**：不同编译器版本或配置

构建字符串通常看起来像：`py311h43a39b2_0`

分解：

- `py311`：Python 版本指示符
- `h43a39b2`：构建配置的哈希值
- `_0`：构建编号

你可以在构建模式中使用通配符来匹配多个构建：

```shell
# Match any CUDA build
pixi add "pytorch=*=cuda*"

# Match Python 3.11 builds
pixi add "numpy=*=py311*"

# Using bracket syntax
pixi add "pytorch [build='cuda*']"
```

### 构建编号

构建编号是一个整数，每次用相同版本重建包时都会递增。
当你需要包的特定重建时使用构建编号约束：

```shell
# Specific build number
pixi add "python [version='3.11.0', build_number='1']"

# Build number constraint
pixi add "numpy [build_number='>=5']"
```

```toml title="pixi.toml"
[dependencies.python]
version = "3.11.0"
build-number = ">=1"
```

构建编号在以下情况下很有用：

- 包被重建以修复编译问题
- 你需要确保你有一个带有错误修复的特定重建
- 使用需要精确构建的可复现环境

### Channel

Channel 是托管 conda 包的仓库。
你可以指定从哪个 channel 获取包：

```shell
# Specific channel by name
pixi add "pytorch [channel='pytorch']"

# Channel URL
pixi add "custom-package [channel='https://prefix.dev/my-channel']"

# Or use the shorter `::` syntax
pixi add pytorch::pytorch
pixi add https://prefix.dev/my-channel::custom-package
```

```toml title="pixi.toml"
[dependencies.pytorch]
channel = "pytorch"

[dependencies.custom-package]
channel = "https://prefix.dev/my-channel"
```

请注意，对于 `pixi add`，channel 也必须在你的 workspace 配置中列出：

```toml title="pixi.toml"
[workspace]
channels = ["conda-forge", "pytorch", "nvidia"]
```

你也可以使用命令行添加这些：

```shell
pixi workspace channel add conda-forge
```

### 校验和（SHA256/MD5）

校验和验证包的完整性和真实性。
将它们用于可复现性和安全性：

```toml title="pixi.toml"
[dependencies.numpy]
version = "1.21.0"
sha256 = "1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
md5 = "abcdef1234567890abcdef1234567890"
```

指定后，Pixi 将：

- 验证下载的包与校验和匹配
- 如果校验和不匹配则安装失败
- 确保你得到你期望的包

出于安全原因，SHA256 优先于 MD5。

### 许可证

指定包的预期许可证。
这对于以下情况很有用：

- 确保符合组织的策略
- 按许可证类型过滤包
- 文档目的

```toml title="pixi.toml"
[dependencies.pytorch]
version = "2.0.*"
license = "BSD-3-Clause"
```

### 文件名

指定要下载的精确包文件名。
这很少需要，但对于以下情况很有用：

- 调试包解析问题
- 确保特定的包产物
- 具有自定义包构建的高级用例

```toml title="pixi.toml"
[dependencies.pytorch]
file-name = "pytorch-2.0.0-cuda.tar.bz2"
```

## 源包

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前会发生变化。
    在为你的 workspace 使用它时请记住这一点。

为了让这些包被识别，它们需要被 Pixi 理解为源包。
查看 [Pixi Manifest 参考](../reference/pixi_manifest.md#the-package-section) 以了解如何在你的 `pixi.toml` 中声明源包。

### 基于路径的源包
```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/package_specifications.toml:path-fields"
```

路径应该是相对于 workspace 根目录或绝对路径，但不鼓励使用绝对路径以保持可移植性。

### 基于 Git 的源包
```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/package_specifications.toml:git-fields"
```

对于 git 仓库，你可以指定：

- **git**：仓库 URL
- **branch**：Git 分支名称
- **tag**：Git 标签
- **rev**：特定 git 修订/SHA
- **subdirectory**：仓库内的路径

## PyPI 包规格

Pixi 还支持使用 `pixi add --pypi` 和在你的 `pixi.toml` 和 `pyproject.toml` 中安装包依赖从 PyPI。
Pixi 实现了标准的 [PEP440 版本说明符](https://peps.python.org/pep-0440/) 来指定包版本。

### 命令行语法

使用 `pixi add --pypi` 时，你可以[类似于 `pip`](https://pip.pypa.io/en/stable/user_guide/#installing-packages)指定包：

```shell
# Simple package
pixi add --pypi requests

# Specific version
pixi add --pypi "requests==2.25.1"

# Version range
pixi add --pypi "requests>=2.20,<3.0"

# Extras
pixi add --pypi "requests[security]==2.25.1"

# URL
pixi add --pypi "requests @ https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl#sha256=2462f94637a34fd532264295e186976db0f5d453d1cdd31473c85a6a161affb6"

# Git repository
pixi add --pypi "requests @ git+https://github.com/psf/requests.git@v2.25.1"
pixi add --pypi requests --git https://github.com/psf/requests.git --tag v2.25.1
pixi add --pypi requests --git https://github.com/psf/requests.git --branch main
pixi add --pypi requests --git https://github.com/psf/requests.git --rev 70298332899f25826e35e42f8d83425124f755a
```

### TOML 映射语法
在你的 `pixi.toml` 或 `pyproject.toml`（在 `[tool.pixi.pypi-dependencies]` 下），你可以像这样指定 PyPI 包：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/package_specifications.toml:pypi-fields"
```

## 进一步阅读

- [Pixi Manifest 参考](../reference/pixi_manifest.md#dependencies) - 完整的依赖规格选项
- [多平台配置](../workspace/multi_platform_configuration.md) - 平台特定的依赖
- [Conda 包规格](https://conda.io/projects/conda/en/latest/user-guide/concepts/pkg-specs.html) - Conda 的包规格文档