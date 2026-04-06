## 传统的 `conda activate` 类激活

如果你更喜欢使用传统的 `conda activate` 类激活，可以使用 `pixi shell-hook` 命令。

```shell
$ which python
python not found
$ eval "$(pixi shell-hook)"
$ (default) which python
/path/to/project/.pixi/envs/default/bin/python
```

例如，对于 `bash` 和 `zsh`，你可以使用以下命令：

```shell
eval "$(pixi shell-hook)"
```

??? tip  "自定义激活函数"
    使用 `--manifest-path` 选项，你还可以指定要激活的环境。如果你想向 `~/.bashrc` 添加一个会激活环境的 `bash` 函数，可以使用以下命令：

    === "Bash/Zsh"
        ```shell
        function pixi_activate() {
            # 如果没有给出路径，默认为当前目录
            local manifest_path="${1:-.}"
            eval "$(pixi shell-hook --manifest-path $manifest_path)"
        }
        ```

        将此函数添加到你的 `~/.bashrc`/`~/.zshrc` 后，你可以通过运行以下命令激活环境：

    === "Fish"

        对于 fish，你也可以评估 `pixi shell-hook` 的输出：

        ```fish
        pixi shell-hook | source
        ```

        或者，如果你想向 `~/.config/fish/config.fish` 添加一个函数：

        ```fish
        function pixi_activate
            # 如果没有给出路径，默认为当前目录
            set -l manifest_path $argv[1]
            test -z "$manifest_path"; and set manifest_path "."

            pixi shell-hook --manifest-path "$manifest_path" | source
        end
        ```
        将此函数添加到你的 `~/.config/fish/config.fish` 后，你可以通过运行以下命令激活环境：

    ```shell
    pixi_activate

    # 或使用特定 manifest
    pixi_activate ~/projects/my_project
    ```


??? tip "使用 direnv"
    有关如何利用 `pixi shell-hook` 与 direnv 集成，请参阅我们的 [direnv 页面](../integration/third_party/direnv.md)。
