`pixi shell` 命令类似于 `conda activate`，但内部工作方式略有不同。
它不需要更改你的 `~/.bashrc` 或其他文件，而是启动一个新的 shell。
这也意味着，不需要 `conda deactivate`，只需退出当前 shell，例如按 `Ctrl+D` 就足够了。

```shell
pixi shell
```

在 Unix 系统上，shell 命令的工作方式是创建一个"假的"PTY 会话来启动 shell，
然后发送一个字符串如 `source /tmp/activation-env-12345.sh` 到 `stdin` 以激活环境。
如果你在 shell 命令的底层查看，你会发现这是在新 shell 会话中执行的第一个东西。

我们生成的临时脚本以 `echo "PIXI_ENV_ACTIVATED"` 结尾，用于检测环境是否成功激活。
如果我们没有在三秒后收到此字符串，我们将向用户发出警告。

## Shell 补全

Shell 补全工具会在 `pixi shell` 中自动加载。
这意味着如果你的环境中的包提供了补全（例如 `git` 或 `cargo`），它们将在没有任何额外配置的情况下可用。

## Pixi Shell 的问题

如前所述，`pixi shell` 只有在启动 shell 后执行激活脚本才能正常工作。
某些在 `~/.bashrc` 中运行的命令可能会吞掉激活命令，环境不会被激活。

例如，如果你的 `~/.bashrc` 包含以下代码，`pixi shell` 几乎不可能成功：

```shell
# 在 WSL 上 - `wsl.exe` 以某种方式接管 `stdin` 并阻止 `pixi shell` 成功
wsl.exe -d wsl-vpnkit --cd /app service wsl-vpnkit start

# 在 macOS 或 Linux 上，一些用户从他们的 `bashrc` 启动 fish 或 nushell
# 如果你想从 bash 启动替代 shell，最好从 `~/.bash_profile` 或 `~/.profile` 做
if [[ $- = *i* ]]; then
  exec ~/.pixi/bin/fish
fi
```

为了解决这个问题，我们建议你按照以下步骤改用 `pixi shell-hook`。

--8<-- "docs/partials/conda-style-activation.md"
