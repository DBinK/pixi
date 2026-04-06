Pixi 默认将你的环境安装到 `.pixi/envs/<env-name>`。如果你想将环境安装到系统上的任意位置，可以使用 [`pixi-install-to-prefix`](https://github.com/pavelzw/pixi-install-to-prefix)。

你可以使用以下命令安装 `pixi-install-to-prefix`：

```bash
pixi global install pixi-install-to-prefix
```

除了全局安装 `pixi-install-to-prefix`，你也可以使用 `pixi exec` 在临时环境中运行 `pixi-install-to-prefix`：

```bash
pixi exec pixi-install-to-prefix ./my-environment
```

```text
Usage: pixi-install-to-prefix [OPTIONS] <PREFIX>

Arguments:
  <PREFIX>  要安装环境的前缀路径

Options:
  -l, --lockfile <LOCKFILE>        pixi lockfile 的路径 [default: pixi.lock]
  -e, --environment <ENVIRONMENT>  要安装的 pixi 环境名称 [default: default]
  -p, --platform <PLATFORM>        要安装的平台 [default: <你的系统平台>]
  -c, --config <CONFIG>            pixi 配置文件的路径。默认不使用配置文件
  -s, --shell <SHELL>              生成激活脚本的 shell。默认：见 README
      --no-activation-scripts      禁用激活脚本的生成
  -v, --verbose...                 增加日志详细程度
  -q, --quiet...                   减少日志详细程度
  -h, --help                       打印帮助
```
