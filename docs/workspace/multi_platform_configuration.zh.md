# 多平台配置

Pixi 的愿景包括在所有主要平台上得到支持。有时需要一些额外的配置才能很好地工作。
在本页面中，你将学习可以配置什么以更好地与你为其构建应用程序的平台保持一致。

这是一个突出某些功能的示例 manifest 文件：

=== "`pixi.toml`"
    ```toml title="pixi.toml"
    [workspace]
    # Default workspace info....
    # A list of platforms you are supporting with your package.
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [dependencies]
    python = ">=3.8"

    [target.win-64.dependencies]
    # Overwrite the needed python version only on win-64
    python = "3.7"


    [activation]
    scripts = ["setup.sh"]

    [target.win-64.activation]
    # Overwrite activation scripts only for windows
    scripts = ["setup.bat"]
    ```
=== "`pyproject.toml`"
    ```toml title="pyproject.toml"
    [tool.pixi.workspace]
    # Default workspace info....
    # A list of platforms you are supporting with your package.
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [tool.pixi.dependencies]
    python = ">=3.8"

    [tool.pixi.target.win-64.dependencies]
    # Overwrite the needed python version only on win-64
    python = "~=3.7.0"


    [tool.pixi.activation]
    scripts = ["setup.sh"]

    [tool.pixi.target.win-64.activation]
    # Overwrite activation scripts only for windows
    scripts = ["setup.bat"]
    ```

## 平台定义

`workspace.platforms` 定义了你的 workspace 支持哪些平台。
当定义多个平台时，Pixi 会单独确定每个平台要安装哪些依赖。
所有这些都存储在锁文件中。

在未配置的平台运行 `pixi install` 会警告用户该平台尚未设置：

```shell
❯ pixi install
 WARN Not installing dependency for (default) on current platform: (osx-arm64) as it is not part of this project's supported platforms.
```

## 目标指定符

使用目标指定符，你可以专门针对单个平台覆盖原始配置。
如果你在目标指定符中针对的目标平台不在 `workspace.platforms` 中指定，Pixi 将抛出错误。

### 依赖

有时你可能只想在特定平台上安装某个依赖，或者你可能想在不同平台上使用不同版本的依赖。

```toml title="pixi.toml"
[dependencies]
python = ">=3.8"

[target.win-64.dependencies]
msmpi = "*"
python = "3.8"
```

在上面的例子中，我们指定我们只在 Windows 上依赖 `msmpi`。
我们还特别希望 在 Windows 上安装时使用 `python 3.8`。
这将覆盖通用依赖集中的依赖。
这不会影响任何其他平台。

你可以使用 pixi 的 cli 将这些依赖添加到 manifest 文件。

```shell
pixi add --platform win-64 posix
```

这也对 `host` 和 `build` 依赖起作用。

```bash
pixi add --host --platform win-64 posix
pixi add --build --platform osx-64 clang
```

结果如下。

```toml title="pixi.toml"
[target.win-64.host-dependencies]
posix = "1.0.0.*"

[target.osx-64.build-dependencies]
clang = "16.0.6.*"
```

### 激活

Pixi 的愿景是实现完全跨平台的 workspace，但通常你需要运行不是你项目构建的工具。
生成的激活脚本通常属于这一类，unix 中的默认脚本是 `bash`，windows 中是 `bat`

为此，你可以使用目标定义来定义你的激活脚本。

```toml title="pixi.toml"
[activation]
scripts = ["setup.sh", "local_setup.bash"]

[target.win-64.activation]
scripts = ["setup.bat", "local_setup.bat"]
```
当此 workspace 在 `win-64` 上使用时，它将只执行目标脚本，而不是默认 `activation.scripts` 中指定的脚本