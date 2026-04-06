除了管理工作流和环境，Pixi 还可以构建包。这在以下场景中非常有用：

- 构建并上传包到 conda 通道（channel）
- 允许用户直接依赖源代码并自动构建它
- 在工作区（workspace）中管理多个包

我们一直在努力通过 Pixi 的 `build` 功能支持这些用例。愿景是能够从源代码构建任意语言、任意平台的包。

!!! note "已知限制"
    目前，`build` 功能有一些限制：

    1. [构建后端](https://github.com/prefix-dev/pixi-build-backends) 数量有限
    2. 构建后端可能缺少很多参数/功能
    3. 工作区依赖不能继承

## 设置项目清单（manifest）

这是使用 `pixi-build` 功能的 Pixi 项目清单（manifest）概述。

项目清单中 `[package]` 部分的更详细概述可在 [项目清单参考](../reference/pixi_manifest.md#the-package-section) 中找到。

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/getting_started/pixi.toml:full"
```

在 `[workspace]` 部分，您可以指定名称、通道（channels）和平台等属性。这目前是 `[project]` 的别名。

由于构建功能仍处于预览阶段，您需要将 "pixi-build" 添加到 `workspace.preview`。

```toml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/getting_started/pixi.toml:preview"
```

在 `package` 中指定要构建的包的特定属性。

```toml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/getting_started/pixi.toml:package"
```

包通过使用构建后端来构建。通过指定 `package.build.backend` 和 `package.build.channels`，您可以确定使用哪个后端以及从哪个通道（channel）下载它。

有[不同的构建后端可用](backends.md)。

Pixi 后端描述了如何为特定语言或构建工具构建 conda 包。在这个例子中，我们使用 `pixi-build-python` 后端来构建 Python 包。

```toml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/getting_started/pixi.toml:build-system"
```

我们需要将我们的包 `python_rich` 作为源代码依赖添加到工作区。

```toml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/getting_started/pixi.toml:dependencies"
```

`python_rich` 使用 `hatchling` 作为 Python 构建后端，因此需要在 `host-dependencies` 中提及。

像 `hatchling` 这样的 Python PEP517 后端知道如何构建 Python 包。因此 `hatchling` 创建一个 Python 包，而 `pixi-build-python` 将 Python 包转换为 conda 包。

在[依赖类型章节](./dependency_types.md#host-dependencies) 中了解更多关于 host-dependencies 的信息。

```toml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/getting_started/pixi.toml:host-dependencies"
```

我们添加 `rich` 作为包的运行依赖。这是必要的，因为包在运行时使用 `rich`。
您可以在[依赖类型章节](./dependency_types.md#dependencies-run-dependencies) 中了解更多关于运行依赖的信息。

```toml
--8<-- "docs/source_files/pixi_workspaces/pixi_build/getting_started/pixi.toml:run-dependencies"
```

## CLI 命令

使用预览功能，您现在可以从源代码构建包。

- 添加了 `pixi build`，它会将您的包构建成 `.conda` 文件。
- 其他命令如 `pixi install` 和 `pixi run` 在存在 `path`、`git` 或 `url` 依赖时会自动使用构建功能。
