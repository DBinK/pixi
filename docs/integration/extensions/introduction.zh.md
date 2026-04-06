# Pixi 扩展

`Pixi` 允许你使用各种扩展来扩展其功能。执行例如 `pixi diff` 时，Pixi 会在你的 PATH 和 pixi 全局目录中搜索可执行文件 `pixi-diff`。然后它将通过任何附加参数执行它。

## 扩展如何工作

Pixi 扩展是遵循简单命名约定的独立可执行文件：它们必须命名为 `pixi-{command}`，其中 `{command}` 是你要添加的子命令的名称。当你运行 `pixi {command}` 时，Pixi 会自动发现并执行相应的 `pixi-{command}` 可执行文件。

例如：

- `pixi diff` → 查找 `pixi-diff` 可执行文件
- `pixi pack` → 查找 `pixi-pack` 可执行文件
- `pixi deploy` → 查找 `pixi-deploy` 可执行文件

## 扩展发现

Pixi 通过按顺序在以下位置搜索带有 `pixi-*` 前缀的可执行文件来发现扩展：

### 1. PATH 环境变量
Pixi 在你的 `PATH` 环境变量的所有目录中搜索带有 `pixi-` 前缀的可执行文件。

### 2. `pixi global` 目录
Pixi 还在 `pixi global` 管理的目录中搜索，这样可以有序地管理扩展而不会弄乱系统 PATH。

当你运行 `pixi --list` 时，所有发现的扩展都会自动与所有内置命令一起列出，使命令易于发现。

## 安装扩展

### 使用 `pixi global`（推荐）

安装 Pixi 扩展的最简单方法是使用 `pixi global install`：

```bash
# 安装单个扩展
pixi global install pixi-pack

# 一次安装多个扩展
pixi global install pixi-pack pixi-diff
```

这种方法有几个优点：

- **隔离的环境**：每个扩展都有自己的环境，防止依赖冲突
- **自动发现**：扩展会自动被 Pixi 发现，无需修改 PATH
- **易于管理**：使用 `pixi global list` 和 `pixi global remove` 管理扩展
- **一致的体验**：扩展出现在 `pixi --list` 中，就像所有内置命令一样，就像 `Cargo` 处理它的方式一样

### 手动安装

你也可以通过将可执行文件放在 PATH 中的任何目录中来手动安装扩展：

```bash
# 下载或构建扩展
curl -L https://github.com/user/pixi-myext/releases/download/v1.0.0/pixi-myext -o pixi-myext
chmod +x pixi-myext
mv pixi-myext ~/.local/bin/
```

## 贡献扩展

### 创建扩展

1. **选择一个描述性名称**：你的扩展应命名为 `pixi-{command}`，其中 `{command}` 清楚地描述其功能。

2. **创建可执行文件**：扩展可以用任何语言编写（Rust、Python、shell 脚本等），只要它们产生可执行的二进制文件。

3. **处理参数**：扩展接收命令名称后传递的所有参数。

### 示例：简单的 Python 扩展

```python
#!/usr/bin/env python3
import sys

def main():
    name = sys.argv[1] if len(sys.argv) > 1 else "World"
    print(f"Hello, {name}!")

if __name__ == "__main__":
    main()
```

将其保存为 `pixi-hello`，使其可执行（`chmod +x pixi-hello`），并放在你的 PATH 中。

用法：`pixi hello Alice` 输出 `Hello, Alice!`

## 最佳实践

- **使用标准参数解析**：像 `clap`（Rust）或 `argparse`（Python）这样的库提供一致的行为
- **支持 `--help`**：用户期望此标准标志
- **遵循 UNIX 约定**：使用退出代码 0 表示成功，非零表示错误
- **与 Pixi 环境配合工作**：扩展应尊重 Pixi 的环境管理

## 命令建议

Pixi 包含由字符串相似性驱动的智能命令建议。如果你拼错了命令名称，Pixi 会从内置命令和可用扩展中建议最接近的匹配项：

```bash
$ pixi pck
error: unrecognized subcommand 'pck`
tip: a similar subcommand exists: 'pack'
```

这适用于内置命令和你安装的任何扩展，使扩展发现变得无缝。

## 获取帮助

- **列出可用扩展**：运行 `pixi --list` 查看所有可用扩展
- **社区**：加入我们的 [Discord](https://discord.gg/kKV8ZxyzY4) 进行讨论和支持

## 另请参阅

- [Pixi Diff](pixi_diff.zh.md) - 比较 lockfile 和环境
- [Pixi Inject](pixi_inject.zh.md) - 将依赖注入到现有环境中
- [Pixi Skills](pixi_skills.zh.md) - 在 LLM 后端之间管理和安装编码代理技能
- [全局工具](../../global_tools/introduction.md) - 管理全局工具安装
