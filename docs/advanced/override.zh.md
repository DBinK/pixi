## 概述

有时我们的直接依赖声明了过时的中间依赖，或者与其他直接依赖的约束太紧。在这种情况下，我们可以在我们的 `pyproject.toml` 或 `pixi.toml` 文件中覆盖中间依赖。

!!! note
    不推荐使用此选项，除非你知道你在做什么，因为 uv 将忽略该依赖的所有版本约束，并使用你指定的版本。

### 覆盖依赖版本
```toml
# pyproject.toml
[tool.pixi.pypi-options.dependency-overrides]
numpy = ">=2.0.0"
```
或在 `pixi.toml` 中：

```toml
# pixi.toml
[pypi-options.dependency-overrides]
numpy = ">=2.0.0"
```
这将覆盖所有依赖使用的 `numpy` 版本至少为 `2.0.0`，无论依赖指定什么。
这在你需要一个与依赖指定的版本不兼容的库版本时很有用。

### 在特定功能中覆盖依赖版本
它也可以在功能级别指定，
```toml
[feature.dev.pypi-options.dependency-overrides]
numpy = ">=2.0.0"
```
这将覆盖 `dev` 功能中所有依赖使用的 `numpy` 版本至少为 `2.0.0`，无论依赖在启用 `dev` 功能时指定什么。

### 与其他覆盖交互

对于特定环境，所有在不同功能中定义的 `dependency-overrides` 将按定义顺序合并。

如果同一个依赖被多次覆盖，我们将使用该环境中**之前的**功能的覆盖。

此外，默认功能将始终出现，并出现在所有覆盖列表的最后。

```toml
# pixi.toml
[pypi-options]
dependency-overrides = { numpy = ">=2.1.0" }

[pypi-dependencies]
numpy = ">=1.25.0"

[feature.dev.pypi-options.dependency-overrides]
numpy = "==2.0.0"

[feature.outdated.pypi-options.dependency-overrides]
numpy = "==1.21.0"

[environments]
dev = ["dev"]
outdated = ["outdated"]
conflict_a=["outdated", "dev"]
conflict_b=["dev","outdated"]
```
以下约束被合并：
default: `numpy >= 2.1.0`
dev: `numpy == 2.0.0`
outdated: `numpy == 1.21.0`
conflict_a: `numpy == 1.21.0`（来自 `outdated`）
conflict_b: `numpy == 2.0.0`（来自 `dev`）

这可能与所有覆盖都应用并合并为结果的直觉形成对比。
这样做是为了避免冲突和混淆：由于用户被授予完全控制覆盖的权利，选择正确的环境覆盖取决于他们自己。
