!!!tip "YouTrack 上的原生 Pixi 支持"
    有一个在 PyCharm 中原生支持 Pixi 的跟踪问题 [PY-79041](https://youtrack.jetbrains.com/issue/PY-79041)。
    如果与你相关，请给它投票。
    对于 CLion，你可以跟踪 [CPP-42761](https://youtrack.jetbrains.com/issue/CPP-42761)。

## PyCharm

<!-- Keep in sync with https://github.com/pavelzw/pixi-pycharm/blob/main/README.md -->

你可以使用 [pixi-pycharm](https://github.com/pavelzw/pixi-pycharm) 包提供的 `conda` shim 将 PyCharm 与 Pixi 环境一起使用。*下面也描述了一种不使用 shim 的[替代方法](#alt-approach)*。

首先，将 `pixi-pycharm` 添加到你的 Pixi 工作区。

```bash
pixi add pixi-pycharm
```

这将确保 conda shim 安装在你的工作区环境中。

安装 `pixi-pycharm` 后，你现在可以配置 PyCharm 使用你的 Pixi 环境。
进入 _Add Python Interpreter_ 对话框（PyCharm 窗口右下角），选择 _Conda Environment_。
将 _Conda Executable_ 设置为 `conda` 文件（Windows 上为 `conda.bat`）的完整路径，该文件位于 `.pixi/envs/default/libexec` 中。
你可以使用以下命令获取路径：

=== "Linux & macOS"
    ```bash
    pixi run 'echo $CONDA_PREFIX/libexec/conda'
    ```

=== "Windows"
    ```bash
    pixi run 'echo $CONDA_PREFIX\\libexec\\conda.bat'
    ```

这是一个可执行文件，它欺骗 PyCharm 认为它是正确的 `conda` 可执行文件。
在后台，它将所有调用重定向到相应的 `pixi` 等效项。

!!!warning "使用此 Pixi 工作区中的 conda shim"
    请确保这是来自此 Pixi 工作区的 conda shim，而不是另一个。
    如果你使用多个 Pixi 工作区，你可能需要相应地调整路径，因为 PyCharm 会记住 conda 可执行文件的路径。

![Add Python Interpreter](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/add-conda-environment-light.png#only-light)
![Add Python Interpreter](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/add-conda-environment-dark.png#only-dark)

选择环境后，PyCharm 现在将使用 Pixi 环境中的 Python 解释器。

PyCharm 现在还应该能够向你显示已安装的包。

![PyCharm package list](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/dependency-list-light.png#only-light)
![PyCharm package list](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/dependency-list-dark.png#only-dark)

你现在可以像往常一样运行程序和测试。

![PyCharm run tests](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/tests-light.png#only-light)
![PyCharm run tests](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/tests-dark.png#only-dark)

<a id="exclude-.pixi"><a/>

!!!tip "将 `.pixi` 标记为排除"
    为了让 PyCharm 不混淆 `.pixi` 目录，请将其标记为排除。

    ![Mark Directory as excluded 1](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-1-light.png#only-light)
    ![Mark Directory as excluded 1](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-1-dark.png#only-dark)
    ![Mark Directory as excluded 2](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-2-light.png#only-light)
    ![Mark Directory as excluded 2](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/mark-directory-as-excluded-2-dark.png#only-dark)

    此外，使用远程解释器时，你应该排除远程机器上的 `.pixi` 目录。
    相反，你应该远程运行 `pixi install` 并从那里选择 conda shim。
    ![Deployment exclude from remote machine](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/deployment-exclude-pixi-light.png#only-light)
    ![Deployment exclude from remote machine](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/deployment-exclude-pixi-dark.png#only-dark)

### 多个环境

如果你的工作区使用[多个环境](../../workspace/multi_environment.md)来测试不同的 Python 版本或依赖，你可以通过在 _Add Python Interpreter_ 对话框中指定 _Use existing environment_ 将多个环境添加到 PyCharm。

![Multiple Pixi environments](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/python-interpreters-multi-env-light.png#only-light)
![Multiple Pixi environments](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/python-interpreters-multi-env-dark.png#only-dark)

然后，你可以在 PyCharm 窗口右下角指定相应的环境。

![Specify environment](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/specify-interpreter-light.png#only-light)
![Specify environment](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/specify-interpreter-dark.png#only-dark)

### 多个 Pixi 工作区

使用多个 Pixi 工作区时，请记住如上所述为每个工作区选择正确的 _Conda Executable_。
你也可能会遇到多个同名环境的情况。

![Multiple default environments](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/multiple-default-envs-light.png#only-light)
![Multiple default environments](https://raw.githubusercontent.com/pavelzw/pixi-pycharm/main/.github/assets/multiple-default-envs-dark.png#only-dark)

建议将环境重命名为唯一的名称。

### 调试

日志写入 `~/.cache/pixi-pycharm.log`。
你可以使用它们来调试问题。
请在[提交错误报告](https://github.com/pavelzw/pixi-pycharm/issues/new?template=bug-report.md)时附上日志。

### 作为可选依赖安装

在某些情况下，你可能只想在本地开发机器上安装 `pixi-pycharm`，而不是在生产环境中。
为此，我们可以使用[多个环境](../../workspace/multi_environment.md)。

```toml
[workspace]
name = "multi-env"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["numpy"]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64"]

[tool.pixi.feature.lint.dependencies]
ruff =  "*"

[tool.pixi.feature.dev.dependencies]
pixi-pycharm = "*"

[tool.pixi.environments]
# 生产环境是默认 Feature 集。
# 添加一个 solve group 以确保在 `default` 和 `prod` 环境中使用相同的版本。
prod = { solve-group = "main" }

# 设置默认环境以包含 dev Feature。
# 通过使用 `default` 而不是 `dev`，你不必在运行 `pixi run` 时指定 `--environment` 标志。
default = { features = ["dev"], solve-group = "main" }

# lint 环境不需要默认 Feature 集，而只需要 `lint` Feature
# 因此也可以从 solve group 中排除。
lint = { features = ["lint"], no-default-feature = true }
```

现在作为用户，你可以运行 `pixi shell`，这将启动默认环境。
在生产环境中，你只需运行 `pixi run -e prod COMMAND`，就会安装最小的 prod 环境。

<a id="alt-approach"></a>
### 使用 environments.txt 的替代方法

还有一种配置 PyCharm 的方法，可以避免使用 pixi-pycharm shim。它要求你在本地安装 conda（如果安装在标准位置，PyCharm 会自动检测到）。

要配置新工作区的解释器：

1. 编辑位于 `~/.conda/environments.txt` 的 conda 环境列表。只需追加你希望包含的任何 pixi 环境的完整文件路径，例如：

    ```
    ...
    /Users/jdoe/my-workspace/.pixi/envs/default
    /Users/jdoe/my-workspace/.pixi/envs/dev
    ```

2. 在 PyCharm 中，为你的工作区添加解释器时，向下滚动到 Python Interpreter 下拉菜单的底部，选择 *Show All ...* 以调出 Python Interpreters 对话框。

3. 选择 `+` 按钮，使用标准 conda 位置添加新的现有 conda 解释器，并从列表中选择所需的前缀。（如果在 PyCharm 运行时编辑了环境文件，你可能需要重新加载环境。）

4. 这将添加环境，但会自动给出与目录路径的最后一个组件匹配的名称，对于 pixi 环境通常只是 `default`。如果你在很多工作区上工作，这尤其成问题。你可以通过单击铅笔图标或使用右键下拉菜单更改 PyCharm 为环境起的名称。

5. 添加并重命名环境后，从列表中选择所需的解释器以在 PyCharm 中使用。

如果你的工作区使用多个环境，你可以通过在 PyCharm 窗口底部的状态栏中选择解释器名称并从列表中选择所需解释器来切换它们。
请注意，这会触发 PyCharm 重新索引，可能不会很快。

与 pixi-pycharm shim 一样，你应该避免使用 PyCharm UI 尝试从你的环境中添加或删除包，并且你应该确保[将 `.pixi` 目录从 PyCharm 索引中排除](#exclude-.pixi)。

## Direnv

要在 [Jetbrains](https://www.jetbrains.com/ides/) 产品中使用 Direnv，首先必须安装 [Direnv 插件](https://plugins.jetbrains.com/plugin/15285-direnv-integration)。
然后按照我们 [Direnv 文档页面](../third_party/direnv.md) 中的说明进行操作。
现在你的 Jetbrains IDE 将在所选的 Pixi 环境中运行。
