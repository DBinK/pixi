`direnv` 是一个工具，当你进入包含你在某时接受的 `.envrc` 文件的目录时，它会自动激活环境。
本教程将演示如何将 `direnv` 与 Pixi 结合使用。

首先，通过运行以下命令安装 `direnv`：

```bash
pixi global install direnv
```

然后在你的 Pixi 工作区根目录中创建包含以下内容的 `.envrc` 文件：

```shell title=".envrc"
watch_file pixi.lock # (1)!
eval "$(pixi shell-hook)" # (2)!
```

1. 这确保每次你的 `pixi.lock` 更改时，`direnv` 都会再次调用 shell-hook。
2. 这会在需要时安装环境并激活它。`direnv` 确保当你离开目录时环境被停用。

```shell
$ cd my-project
direnv: error /my-project/.envrc is blocked. Run `direnv allow` to approve its content
$ direnv allow
direnv: loading /my-project/.envrc
✔ Project in /my-project is ready to use!
direnv: export +CONDA_DEFAULT_ENV +CONDA_PREFIX +PIXI_ENVIRONMENT_NAME +PIXI_ENVIRONMENT_PLATFORMS +PIXI_PROJECT_MANIFEST +PIXI_PROJECT_NAME +PIXI_PROJECT_ROOT +PIXI_PROJECT_VERSION +PIXI_PROMPT ~PATH
$ which python
/my-project/.pixi/envs/default/bin/python
$ cd ..
direnv: unloading
$ which python
python not found
```

虽然 `direnv` 带有[常见 shell 的钩子](https://direnv.net/docs/hook.html)，但在 IDE 中使用这些 shell 钩子时不应依赖它们。

在这里你可以看到如何为你的编辑器设置 `direnv`：

- [VSCode](../editor/vscode.md#direnv-extension)
- [Jetbrains](../editor/jetbrains.md#direnv)
- [Zed](../editor/zed.md)
