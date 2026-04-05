# 全局工具

<div style="text-align:center">
  <video autoplay muted loop>
   <source src="https://github.com/user-attachments/assets/e94dc06f-75ae-4aa0-8830-7cb190d3f367" type="video/webm" />
   <p>Pixi global demo</p>
  </video>
</div>

使用 `pixi global`，用户可以管理全局安装的工具，使它们可以从任何目录访问。
这意味着 Pixi 环境将被放置在全局位置，并且工具将暴露给系统 `PATH`，允许你从命令行运行它们。
某些包，特别是具有图形用户界面的包，也将添加开始菜单条目。

## 基本用法

运行以下命令在系统上安装 [`rattler-build`](https://prefix-dev.github.io/rattler-build/latest/)。

```bash
pixi global install rattler-build
```

`pixi global` 的优点是，默认情况下，它将每个包隔离在它自己的环境中，只暴露必要的入口点。
这意味着你不必担心删除一个包并意外破坏看似不相关的包。
这个行为与 [`pipx`](https://pipx.pypa.io/latest/installation/) 非常相似。

但是，有时你可能希望在同一个环境中有多个依赖项。
例如，虽然 `ipython` 本身非常有用，但当使用它时 `numpy` 和 `matplotlib` 可用时，它会变得更加有用。

让我们执行以下命令：

```bash
pixi global install ipython --with numpy --with matplotlib
```

`numpy` 暴露可执行文件，但由于它是通过 `--with` 添加的，其可执行文件不会被暴露。

现在导入 `numpy` 和 `matplotlib` 如预期般工作。

```bash
ipython -c 'import numpy; import matplotlib'
```

有时，你可能希望在系统上安装同一包的多个版本。
由于它们都将在系统 `PATH` 上可用，它们需要以不同的名称暴露。

让我们看看以下命令：

```bash
pixi global install --expose py3=python "python=3.12"
```

通过指定 `--expose`，我们指定我们希望将 `python` 可执行文件以名称 `py3` 暴露。
`python` 包有更多可执行文件，但由于我们指定了 `--exposed`，它们不会自动暴露。

你可以运行 `py3` 来启动 python 解释器。

```shell
py3 -c "print('Hello World')"
```

## 从源码安装

Pixi global 还允许你安装 [Pixi 包](../build/getting_started.md)。
假设我们有一个 C++ 包想从源码全局安装。
首先，它需要一个包 manifest：

```toml title="pixi.toml"
[package]
name = "cpp_math"
version = "0.1.0"

[package.build]
backend = { name = "pixi-build-cmake", version = "*" }
```

如果源码在你的机器上，你可以像这样安装它：

```shell
pixi global install --path /path/to/cpp_math
```

如果源码在 git 仓库中，你可以像这样访问它：

```shell
pixi global install --git https://github.com/ORG_NAME/cpp_math.git
```

如果源码包含多个输出，必须注意，请参阅例如这个配方：

```yaml title="recipe.yaml"
recipe:
  name: multi-output
  version: "0.1.0"

outputs:
  - package:
      name: foobar
    build:
      script:
        - if: win
          then:
            - if not exist %PREFIX%\bin mkdir %PREFIX%\bin
            - echo @echo off > %PREFIX%\bin\foobar.bat
            - echo echo Hello from foobar >> %PREFIX%\bin\foobar.bat
          else:
            - mkdir -p $PREFIX/bin
            - echo "#!/usr/bin/env bash" > $PREFIX/bin/foobar
            - echo "echo Hello from foobar" >> $PREFIX/bin/foobar
            - chmod +x $PREFIX/bin/foobar

  - package:
      name: bizbar
    build:
      script:
        - if: win
          then:
            - if not exist %PREFIX%\bin mkdir %PREFIX%\bin
            - echo @echo off > %PREFIX%\bin\bizbar.bat
            - echo echo Hello from bizbar >> %PREFIX%\bin\bizbar.bat
          else:
            - mkdir -p $PREFIX/bin
            - echo "#!/usr/bin/env bash" > $PREFIX/bin/bizbar
            - echo "echo Hello from bizbar" >> $PREFIX/bin/bizbar
            - chmod +x $PREFIX/bin/bizbar
```

在这种情况下，我们必须指定我们想要安装哪个输出：

```shell
pixi global install --path /path/to/package foobar
```

## Shell 补全

当你在终端中工作时，你在使用 shell，shell 可以处理命令行工具的补全。
工作原理是这样的：你在终端中输入 "git -" 然后按 `<TAB>`。
然后，你的 shell 会向你展示 `git` 提供所有标志。
但是，只有当你安装了相关工具的补全时这才有效。
如果通过 `pixi global` 安装的工具包含补全，它们将自动安装。
目前，仅支持 Linux 和 macOS。

首先用 `pixi global` 安装一个工具：

```shell
pixi global install git
```

补全可以在 [`$PIXI_HOME`](../reference/environment_variables.md)`/completions` 下找到。

然后你可以在 shell 的启动脚本中加载补全：

```bash title="~/.bashrc"
# bash, default on most Linux distributions
for file in ~/.pixi/completions/bash/*; do
    [ -e "$file" ] && source "$file"
done
```

```zsh title="~/.zshrc"
# zsh, default on macOS
fpath+=(~/.pixi/completions/zsh)
autoload -Uz compinit
compinit
```

```fish title="~/.config/fish/config.fish"
# fish
for file in ~/.pixi/completions/fish/*
    source $file
end
```

!!! note

    只要它们的二进制文件以相同的名称暴露，包的补全就会安装：例如 `exposed = { git = "git" }`。

!!! tip

    你最喜欢的 CLI 工具缺少 shell 补全吗？类似 [conda-forge/crush-feedstock #37](https://github.com/conda-forge/crush-feedstock/pull/37/changes) 添加它们。

## 一次添加一系列工具

不指定安装环境，你可以一次添加多个工具：

```shell
pixi global install pixi-pack rattler-build
```

此命令在 manifest 中生成以下条目：

```toml
[envs.pixi-pack]
channels = ["conda-forge"]
dependencies= { pixi-pack = "*" }
exposed = { pixi-pack = "pixi-pack" }

[envs.rattler-build]
channels = ["conda-forge"]
dependencies = { rattler-build = "*" }
exposed = { rattler-build = "rattler-build" }
```

创建两个互不干扰的环境，同时只暴露最少需要的二进制文件。

## 创建数据科学沙盒环境

你可以使用以下命令创建包含多个工具的环境：

```shell
pixi global install --environment data-science --expose jupyter --expose ipython jupyter numpy pandas matplotlib ipython
```

此命令在 manifest 中生成以下条目：

```toml
[envs.data-science]
channels = ["conda-forge"]
dependencies = { jupyter = "*", ipython = "*" }
exposed = { jupyter = "jupyter", ipython = "ipython" }
```

在这个设置中，`jupyter` 和 `ipython` 都从 `data-science` 环境暴露，允许你运行：

```shell
> ipython
# Or
> jupyter lab
```

这些命令将全局可用，使你更容易访问你喜欢的工具，而无需切换环境。

## 为不同平台安装包

你可以使用 `--platform` 标志为不同平台安装包。
当你想要为不同平台安装包时很有用，例如在 `osx-arm64` 上安装 `osx-64` 包。
例如，在 `osx-arm64` 上运行：

```shell
pixi global install --platform osx-64 python
```

将在 manifest 中创建以下条目：

```toml
[envs.python]
channels = ["conda-forge"]
platforms = ["osx-64"]
dependencies = { python = "*" }
# ...
```

## 打包

### 选择退出 `CONDA_PREFIX`

Pixi 在运行全局暴露的可执行文件之前激活目标环境，这通常会将 `CONDA_PREFIX` 设置为该环境的路径。
一些工具检查 `CONDA_PREFIX` 并期望它指向标准 Conda 安装，当工具从 Pixi 管理的前缀运行时可能导致混淆的行为。

包作者可以通过在环境中的 `etc/pixi/<executable>/global-ignore-conda-prefix` 发送标记文件来选择不导出 `CONDA_PREFIX`（例如，对于名为 `borg` 的可执行文件，为 `etc/pixi/borg/global-ignore-conda-prefix`）。
当此文件存在用于暴露的可执行文件时，Pixi 会从环境变量中移除 `CONDA_PREFIX`，
让工具表现得好像没有 Conda 环境处于活动状态。

这是一个最小的 `recipe.yaml` 片段，在使用可执行文件 `borg` 构建包时添加标记：

```yaml
build:
  script:
    - mkdir -p $PREFIX/etc/pixi/borg
    - touch $PREFIX/etc/pixi/borg/global-ignore-conda-prefix
```

在使用 `pixi global install` 安装这样的包后，暴露的可执行文件不再看到 `CONDA_PREFIX` 并可以回退到其默认行为。