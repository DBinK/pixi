# 更新日志

本项目的所有显着变化都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)，
并遵循 [Semantic Versioning](https://semver.org/spec/v2.0.0.html)。

### [0.66.0] - 2026-03-16
#### ✨ 亮点

想要类似 `conda`/`mamba` 的 Pixi 工作区工作流程？此版本带来了注册工作区！

你现在可以注册你的 Pixi 工作区，然后可以使用其名称从机器上的任何位置引用它。例如：

```shell
cd path/to/a/pixi/workspace
pixi workspace register
cd
pixi run --workspace workspace-name task-name
pixi shell -w workspace-name
pixi shell-hook -w workspace-name
```

你还可以为可能属于你的环境而不需要明确要求的包指定 `constraints`（类似于 Conda 包的 `run_constraints`）：

```toml
[constraints]
openssl = ">=3"
```

#### 新增

- 添加 `constraints` 以按 @delsner 在 [#5603](https://github.com/prefix-dev/pixi/pull/5603) 限制依赖版本
- `pixi search` 改进：允许任意 MatchSpecs，添加 `--json` 按 @pavelzw 在 [#5442](https://github.com/prefix-dev/pixi/pull/5442)
- 添加原子写入工具以进行安全文件操作按 @baszalmstra 在 [#5500](https://github.com/prefix-dev/pixi/pull/5500)
- 支持 `--` 分隔符以将额外参数传递给类型化参数任务按 @ruben-arts 在 [#5569](https://github.com/prefix-dev/pixi/pull/5569)
- 同样为任务中的环境变量添加模板参数按 @tdejager 在 [#5613](https://github.com/prefix-dev/pixi/pull/5613)
- 允许用户通过注册表使用 Pixi 命名工作区按 @soapy1 在 [#5277](https://github.com/prefix-dev/pixi/pull/5277)
- 添加 `pyproject.toml` 模式按 @bollwyvl 在 [#5583](https://github.com/prefix-dev/pixi/pull/5583)

#### 文档

- 添加 `pixi-browse` 按 @pavelzw 在 [#5642](https://github.com/prefix-dev/pixi/pull/5642) 和 [#5664](https://github.com/prefix-dev/pixi/pull/5664)
- 重写 lock file 文档页面按 @VeckoTheGecko 在 [#5404](https://github.com/prefix-dev/pixi/pull/5404)
- 修复 pixi-build 文档中关于 uv 在安装程序选择中的问题按 @kilian-hu 在 [#5670](https://github.com/prefix-dev/pixi/pull/5670)
- 添加 prefix.dev 部署文档按 @wolfv 在 [#5671](https://github.com/prefix-dev/pixi/pull/5671)
- 修复 pyproject.toml 设置文档中的拼写错误按 @LiamConnors 在 [#5620](https://github.com/prefix-dev/pixi/pull/5620)

#### 修复

- 允许在同一 solve group 中不同平台求解环境按 @borchero 在 [#5538](https://github.com/prefix-dev/pixi/pull/5538)
- 错误合并的竞态条件按 @baszalmstra 在 [#5591](https://github.com/prefix-dev/pixi/pull/5591)
- 支持 pyproject.toml 解析中的隐式表语法按 @suleman1412 在 [#5580](https://github.com/prefix-dev/pixi/pull/5580)
- 将 `aws-lc-sys` 更新到 0.38.0 按 @baszalmstra 在 [#5610](https://github.com/prefix-dev/pixi/pull/5610)
- 发布中的哈希计算按 @wolfv 在 [#5605](https://github.com/prefix-dev/pixi/pull/5605)
- pypi 安装期间的可编辑性检查按 @tdejager 在 [#5617](https://github.com/prefix-dev/pixi/pull/5617)
- 避免当前工作目录不再存在时出现 panic 按 @mohitdebian 在 [#5652](https://github.com/prefix-dev/pixi/pull/5652)
- 包名包含点时的缓存文件路径损坏按 @ytausch 在 [#5668](https://github.com/prefix-dev/pixi/pull/5668)
- `pixi clean` 行为改进按 @ruben-arts 在 [#5685](https://github.com/prefix-dev/pixi/pull/5685)
- 对 `pixi_progress::println!` 使用 `mp.suspend` 按 @lucascolley 在 [#5459](https://github.com/prefix-dev/pixi/pull/5459)

#### 新贡献者
* @soapy1 在 [#5277](https://github.com/prefix-dev/pixi/pull/5277) 做了第一次贡献
* @mohitdebian 在 [#5652](https://github.com/prefix-dev/pixi/pull/5652) 做了第一次贡献
* @suleman1412 在 [#5580](https://github.com/prefix-dev/pixi/pull/5580) 做了第一次贡献

### [0.65.0] - 2026-02-26
#### ✨ 亮点

我们现在正确签署我们自己的 Windows 二进制文件，不再出现"智能屏幕"的错误。

#### 新增

- 为 `pixi-gui` 添加 list_packages 按 @haffer-felix 在 [#4930](https://github.com/prefix-dev/pixi/pull/4930)
- 为任务添加 `choices`，带有参数验证按 @Hofer-Julian 在 [#5543](https://github.com/prefix-dev/pixi/pull/5543)
- 使用签名构建新的 trampoline 二进制文件，按 @wolfv 在 [#5523](https://github.com/prefix-dev/pixi/pull/5523)
- 为 `pixi global list` 添加 `--json` 标志按 @Hofer-Julian 在 [#5530](https://github.com/prefix-dev/pixi/pull/5530)

#### 文档

- 改进侧边栏渲染按 @Hofer-Julian 在 [#5550](https://github.com/prefix-dev/pixi/pull/5550)
- 改进 shell 补全文档按 @pavelzw 在 [#5546](https://github.com/prefix-dev/pixi/pull/5546)

#### 修复

- `tool.pixi.pypi-options.no-build = true` 与 `--locked` 组合按 @hameerabbasi 在 [#5554](https://github.com/prefix-dev/pixi/pull/5554)

### [0.64.0] - 2026-02-23
#### ✨ 亮点

大型版本，包含大量不同的修复和小功能，但这次没有总体主题。

#### 新增

- 为 `pixi lock` 命令添加 `--dry-run` 标志按 @akshatsrivastava11 在 [#5288](https://github.com/prefix-dev/pixi/pull/5288)
- 为 `pixi run` 添加 `--executable` 标志按 @kajal-jotwani 在 [#5253](https://github.com/prefix-dev/pixi/pull/5253)
