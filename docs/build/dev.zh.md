# 开发依赖

`[dev]` 表中的源码包不会被构建或安装到 pixi 环境中。
这些包的 `build-dependencies`、`host-dependencies` 和 `run-dependencies` 会安装到 pixi 环境中。

`[dependencies]` 部分中的源码依赖会在位于 `.pixi/build` 的隔离环境中构建，然后将生成的 conda 包安装到默认环境中。
这意味着 `build-` 和 `host-dependencies` 不会在 pixi 环境中。

本文档解释如何使用 `[dev]` 表来依赖包的开发依赖。

## 使用 `[dev]` 表

假设有一个你想使用 Pixi 开发的 Rust 包。
然后我们添加一个 `pixi.toml` manifest 文件：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/dev/pixi.toml:minimal"
```

现在你可以使用 Pixi 将包构建为 conda 包：

```bash
pixi build
```

由于隔离，开发依赖（如 `cargo`）在 `pixi run` 中不可用。

要更改这一点，你可以在 manifest 文件中添加一个 `[dev]` 表：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/dev/pixi.toml:dev"
```

现在当你运行 `pixi install` 时，开发依赖将安装到 Pixi 环境中。
这意味着你现在可以在 `pixi run` 中使用 `cargo`：

```bash
pixi run cargo run
```

这是因为 `[dev]` 表中的包不会被构建或安装，但它们的 `build-`、`host-`、`run-dependencies` 都会。
因此，你可以在开发过程中使用它们。

## 扩展示例
这是一个使用 `[dev]` 表的完整 `pixi.toml` 示例：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/pixi_build/dev/pixi.toml"
```

当你运行 `pixi list` 时，你会看到你安装了 `cmake`、`python`、`bat` 和 `rust`，而没有在实际依赖项中定义它们。
这是因为它们定义在包含在 `[dev]` 表中的包的依赖项中。