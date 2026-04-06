
## `conda`、`mamba`、`poetry`、`pip` 有什么区别？

| 工具   | 安装 Python | 构建包 | 运行预定义任务 | 内置 lock file | 快速 | 无 Python 使用 |
|--------|-------------|--------|----------------|---------------|------|---------------|
| Conda  | ✅          | ❌     | ❌             | ❌             | ❌   | ❌            |
| Mamba  | ✅          | ❌     | ❌             | ❌             | ✅   | ✅(https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html) |
| Pip    | ❌          | ✅     | ❌             | ❌             | ❌   | ❌            |
| Pixi   | ✅          | 🚧     | ✅             | ✅             | ✅   | ✅            |
| Poetry | ❌          | ✅     | ❌             | ✅             | ❌   | ❌            |

## 为什么叫 `pixi`

从 `prefix` 这个名字开始，我们迭代直到有一个容易发音、拼写和记住的名字。
当时也没有使用该名称的 CLI 工具。
不像 `px`、`pex`、`pax` 等。
在代码模式下我们这样拼写 `pixi`，否则我们总是以大写字母开头：Pixi。
我们认为这个名字激发了好奇心和乐趣，如果你不同意，我很抱歉，但你总是可以把它别名为你喜欢的任何名字。

=== "Linux & macOS"
    ```shell
    alias not_pixi="pixi"
    ```
=== "Windows"
    PowerShell:
    ```powershell
    New-Alias -Name not_pixi -Value pixi
    ```

## `pixi build` 在哪里

**TL;DR**：它即将推出，我们保证！

`pixi build` 将是能够从 Pixi 工作区生成 conda 包的子命令。
这需要一个可靠的构建工具，我们正在创建 [`rattler-build`](https://github.com/prefix-dev/rattler-build)，它将作为 pixi 中的库使用。
