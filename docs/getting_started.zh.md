# Pixi 基础使用

Pixi 可以做很多事情，但设计初衷是简单易用。
让我们来了解一下 Pixi 的基础用法。

## 管理 workspace

- [`pixi init`](./reference/cli/pixi/init.md) - 在当前目录创建新的 Pixi manifest
- [`pixi add`](./reference/cli/pixi/add.md) - 向 manifest 添加依赖
- [`pixi remove`](./reference/cli/pixi/remove.md) - 从 manifest 移除依赖
- [`pixi update`](./reference/cli/pixi/update.md) - 更新 manifest 中的依赖
- [`pixi upgrade`](./reference/cli/pixi/upgrade.md) - 将 manifest 中的依赖升级到最新版本，即使你固定到了特定版本
- [`pixi lock`](./reference/cli/pixi/lock.md) - 为 manifest 创建或更新锁文件
- [`pixi info`](./reference/cli/pixi/info.md) - 显示关于 workspace 的信息
- [`pixi run`](./reference/cli/pixi/run.md) - 运行 manifest 中定义的任务或当前环境中的任何命令
- [`pixi shell`](./reference/cli/pixi/shell.md) - 在当前环境中启动 shell
- [`pixi list`](./reference/cli/pixi/list.md) - 列出当前环境中的所有依赖
- [`pixi tree`](./reference/cli/pixi/tree.md) - 显示当前环境中依赖的树状结构
- [`pixi clean`](./reference/cli/pixi/clean.md) - 从你的机器上移除环境

## 管理全局安装

Pixi 可以管理全局环境中的全局工具安装。
它将环境安装在一个中心位置，因此你可以从任何地方使用它们。

- [`pixi global install`](./reference/cli/pixi/global/install.md) - 将包安装到 global space 中自己的环境
- [`pixi global uninstall`](./reference/cli/pixi/global/uninstall.md) - 从 global space 卸载环境
- [`pixi global add`](./reference/cli/pixi/global/add.md) - 将包添加到现有全局环境
- [`pixi global sync`](./reference/cli/pixi/global/sync.md) - 将全局安装的环境与全局 manifest 同步，描述你想要安装的所有环境
- [`pixi global edit`](./reference/cli/pixi/global/edit.md) - 编辑全局 manifest
- [`pixi global update`](./reference/cli/pixi/global/update.md) - 更新全局环境
- [`pixi global list`](./reference/cli/pixi/global/list.md) - 列出所有全局环境

更多信息：[全局工具](./global_tools/introduction.md)

## 运行一次性命令

Pixi 可以在特定环境中运行一次性命令。

- [`pixi exec`](./reference/cli/pixi/exec.md) - 在临时环境中运行命令
- [`pixi exec --spec`](./reference/cli/pixi/exec.md#arg---spec)   - 在具有特定规格的临时环境中运行命令

例如：

```bash
> pixi exec python -VV
Python 3.13.5 | packaged by conda-forge | (main, Jun 16 2025, 08:24:05) [Clang 18.1.8 ]
> pixi exec --spec "python=3.12" python -VV
Python 3.12.11 | packaged by conda-forge | (main, Jun  4 2025, 14:38:53) [Clang 18.1.8 ]
```

## 多环境

Pixi workspace 允许你管理多个环境。环境由一个或多个 Feature 组成。

- [`pixi add --feature`](./reference/cli/pixi/add.md#arg---feature) - 向 Feature 添加包
- [`pixi task add --feature`](./reference/cli/pixi/task/add.md#arg---feature) - 向特定 Feature 添加任务
- [`pixi workspace environment add`](./reference/cli/pixi/workspace/environment/add.md) - 向 workspace 添加环境
- [`pixi run --environment`](./reference/cli/pixi/run.md#arg---environment) - 在特定环境中运行命令
- [`pixi shell --environment`](./reference/cli/pixi/shell.md#arg---environment) - 激活特定环境
- [`pixi list --environment`](./reference/cli/pixi/list.md#arg---environment) - 列出特定环境中的依赖

更多信息：[多环境](./workspace/multi_environment.md)

## 任务

Pixi 可以使用其内置的任务运行器运行跨平台任务。
这可以是预定义的任务或任何普通可执行文件。

- [`pixi run`](./reference/cli/pixi/run.md) - 运行任务或命令
- [`pixi task add`](./reference/cli/pixi/task/add.md) - 向 manifest 添加新任务

任务可以依赖其他任务。
以下是更复杂的任务用例示例

```toml title="pixi.toml"
[tasks]
build = "make build"
# using the toml table view
[tasks.test]
cmd = "pytest"
depends-on = ["build"]
```

更多信息：[任务](./workspace/advanced_tasks.md)

## 多平台支持

Pixi 开箱即用地支持多个平台。
你可以指定你的 workspace 支持哪些平台，Pixi 将确保依赖与这些平台兼容。

- [`pixi add --platform`](./reference/cli/pixi/add.md#arg---platform) - 仅向特定平台添加包
- [`pixi workspace platform add`](./reference/cli/pixi/workspace/platform/add.md) - 向 workspace 添加你想要支持的平台

更多信息：[多平台支持](./workspace/multi_platform_configuration.md)

## 实用工具

Pixi 带有一组实用工具来帮助你调试或管理你的设置。

- [`pixi info`](./reference/cli/pixi/info.md) - 显示当前 workspace 和全局设置的信息
- [`pixi config`](./reference/cli/pixi/config.md) - 显示或编辑 Pixi 配置
- [`pixi tree`](./reference/cli/pixi/tree.md) - 显示当前环境中依赖的树状结构
- [`pixi list`](./reference/cli/pixi/list.md) - 列出当前环境中的所有依赖
- [`pixi clean`](./reference/cli/pixi/clean.md) - 从你的机器上移除 workspace 环境
- `pixi help` - 显示 Pixi 命令的帮助
- `pixi help <subcommand>` - 显示特定 Pixi 命令的帮助
- [`pixi auth`](./reference/cli/pixi/auth.md) - 管理 conda channel 的认证
- [`pixi search`](./reference/cli/pixi/search.md) - 在配置的 channel 中搜索包
- [`pixi completion`](./reference/cli/pixi/completion.md) - 为 Pixi 命令生成 shell 补全脚本

## 继续深入

Pixi 还有更多功能待你探索。
查看左侧边栏上的主题以了解更多。

别忘了 [加入我们的 Discord](https://discord.gg/kKV8ZxyzY4) 来参与 Pixi 爱好者社区！