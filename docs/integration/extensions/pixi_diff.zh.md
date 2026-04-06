![pixi-diff demo](https://raw.githubusercontent.com/pavelzw/pixi-diff/refs/heads/main/.github/assets/demo/demo-light.gif#only-light)
![pixi-diff demo](https://raw.githubusercontent.com/pavelzw/pixi-diff/refs/heads/main/.github/assets/demo/demo-dark.gif#only-dark)

你可能想知道在 pull request 中反复添加和删除依赖后，lockfile 中发生了什么变化。
为此，你可以使用 [pavelzw/pixi-diff](https://github.com/pavelzw/pixi-diff) 来计算两个 lockfile 之间的差异。
这可以与 [pavelzw/pixi-diff-to-markdown](https://github.com/pavelzw/pixi-diff-to-markdown) 结合使用，生成一个以人类可读格式显示差异的 markdown 文件。
使用 [charmbracelet/glow](https://github.com/charmbracelet/glow)，你甚至可以在终端中呈现 markdown 文件。

!!!tip "全局安装工具"
    上述所有工具都可以在 conda-forge 上获取，可以使用 [`pixi global install`](../../global_tools/introduction.md) 安装。

    ```bash
    pixi global install pixi-diff pixi-diff-to-markdown glow-md
    ```

`pixi diff --before pixi.lock.old --after pixi.lock.new` 将输出一个包含两个 lockfile 之间差异的 JSON 对象，类似于 [`pixi update --json`](../../reference/cli/pixi/update.md)。

```bash
$ pixi diff --before pixi.lock.old --after pixi.lock.new
{
  "version": 1,
  "environment": {
    "default": {
      "osx-arm64": [
        {
          "name": "libmpdec",
          "before": null,
          "after": {
            "conda": "https://conda.anaconda.org/conda-forge/osx-arm64/libmpdec-4.0.0-h99b78c6_0.conda",
            "sha256": "f7917de9117d3a5fe12a39e185c7ce424f8d5010a6f97b4333e8a1dcb2889d16",
            "md5": "7476305c35dd9acef48da8f754eedb40",
            "depends": [
              "__osx >=11.0"
            ],
            "license": "BSD-2-Clause",
            "license_family": "BSD",
            "size": 69263,
            "timestamp": 1723817629767
          },
          "type": "conda"
        },
// ...
```

命名管道可用于比较 git 历史中不同状态的 lockfile：

```bash
# bash / zsh
pixi diff --before <(git show HEAD~20:pixi.lock) --after pixi.lock

# fish
pixi diff --before (git show HEAD~20:pixi.lock | psub) --after pixi.lock
```

或者通过 stdin 指定"before"或"after" lockfile：

```bash
git show HEAD~20:pixi.lock | pixi diff --before - --after pixi.lock
```

这可以与 [`pixi-diff-to-markdown`](https://github.com/pavelzw/pixi-diff-to-markdown) 集成，生成以人类可读格式显示差异的 markdown 文件：

```bash
pixi diff <(git show HEAD~20:pixi.lock) pixi.lock | pixi diff-to-markdown > diff.md
```

!!!tip "GitHub Actions 中的 pixi-diff-to-markdown 更新"
    有关 [`pixi-diff-to-markdown`](https://github.com/pavelzw/pixi-diff-to-markdown) 的其他用法，另请参阅我们关于[使用 GitHub Actions 更新 lockfile](../ci/updates_github_actions.md) 的页面。

你可以使用 [`glow`](https://github.com/charmbracelet/glow) 在终端中查看生成的 markdown 文件。

```bash
glow diff.md --tui
```

你也可以使用 [`glow`](https://github.com/charmbracelet/glow) 直接从 stdin 查看 markdown 文件。

```bash
pixi diff <(git show HEAD~20:pixi.lock) pixi.lock | pixi diff-to-markdown | glow --tui
```
