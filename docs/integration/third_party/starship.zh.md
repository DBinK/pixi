![Starship with Pixi support](../../assets/starship-light.png#only-light)
![Starship with Pixi support](../../assets/starship-dark.png#only-dark)

[Starship](https://starship.rs) 是一个面向开发者的跨平台和跨 shell 提示符，类似于 oh-my-zsh，但专注于性能和简洁。它也完全支持 Pixi。你可以使用以下命令安装它：

```bash
pixi global install starship
```

!!!tip ""
    有关如何配置和设置 starship 的信息，请参阅[官方文档](https://starship.rs/config/#pixi)。

为了让 starship 始终找到正确的 python 可执行文件，你可以调整其配置文件。

```toml title="~/.config/starship.toml"
[python]
# 为 pixi 自定义 python 二进制路径
python_binary = [
  # 如果在 pixi shell 中，这是来自 PATH 的 python
  # （假设你的全局 PATH 上没有 python）
  "python",
  # 如果 pixi 的 python 可用则回退到它
  ".pixi/envs/default/bin/python",
]
```

默认情况下，starship 使用 🧚🏻 作为 pixi 的符号。如果你想使用不同的符号，可以按如下方式调整：

```toml title="~/.config/starship.toml"
[pixi]
symbol = "📦 "
```

由于 starship 已经在 pixi 环境激活时显示自定义消息，你可以禁用 pixi 的自定义 PS1：

```plaintext
pixi config set shell.change-ps1 "false"
```
