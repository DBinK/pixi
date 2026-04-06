[pixi-skills](https://github.com/pavelzw/pixi-skills) 跨多个 LLM 后端管理和安装编码代理技能。

它发现打包在 pixi 环境中的技能，并通过符号链接将它们安装到各种编码代理的配置目录中。

有关 pixi-skills 动机和设计的详细解释，请参阅博客文章[使用包管理器管理代理技能](https://pavel.pink/blog/pixi-skills)。

## 安装

```bash
pixi global install pixi-skills
```

## 概念

### 技能

技能是包含带有 YAML frontmatter 的 `SKILL.md` 文件的目录：

```markdown
---
name: my-skill
description: "Does something useful for the agent"
---

Skill instructions go here as Markdown.
The agent reads this file to understand what the skill does.
```

`name` 字段是可选的，默认为目录名称。
`description` 字段是必填的。

### skill-forge

可在 [skill-forge](https://prefix.dev/channels/skill-forge) 通道上作为 conda 包获取的即用型技能集合（[source](https://github.com/pavelzw/skill-forge)）。

要使用 skill-forge 的技能，将通道和所需的技能包添加到你的 `pixi.toml`：

```toml
[workspace]
channels = ["conda-forge", "https://prefix.dev/skill-forge"]

[dependencies]
polars = ">=1,<2"

[feature.dev.dependencies]
agent-skill-polars = "*"
```

### 作用域

- **本地**技能从当前项目的 pixi 环境 `.pixi/envs/<env>/share/agent-skills/` 中发现。
- **全局**技能从全局安装的 pixi 包 `~/.pixi/envs/agent-skill-*/share/agent-skills/` 中发现。

技能作为相对符号链接安装以提高可移植性。

## 用法

### 交互式管理技能

```bash
# 交互模式 - 提示选择后端和作用域
pixi skills manage

# 直接指定后端和作用域
pixi skills manage --backend claude --scope local
```

这将打开一个交互式复选框选择器，你可以在其中选择要安装或卸载的技能。

### 列出可用技能

```bash
# 列出所有本地和全局技能
pixi skills list

# 仅列出本地技能
pixi skills list --scope local

# 列出特定 pixi 环境中的技能
pixi skills list --env myenv
```

### 显示已安装的技能

```bash
# 显示所有后端中已安装的技能
pixi skills status

# 显示特定后端中已安装的技能
pixi skills status --backend claude
```
