你可以将 GitHub Actions 与 [pavelzw/pixi-diff-to-markdown](https://github.com/pavelzw/pixi-diff-to-markdown) 结合使用，自动更新 lockfile，类似于其他生态系统中的 dependabot 或 renovate。

![Update lockfiles](../../assets/update-lockfile-light.png#only-light)
![Update lockfiles](../../assets/update-lockfile-dark.png#only-dark)

!!!note "Dependabot/Renovate 对 pixi 的支持"
    你可以在 [dependabot/dependabot-core #2227](https://github.com/dependabot/dependabot-core/issues/2227#issuecomment-1709069470) 跟踪对 pixi 的原生 Dependabot 支持。

## 如何使用

首先，在你的仓库中创建一个新的 GitHub Actions workflow 文件。

```yaml title=".github/workflows/update-lockfiles.yml"
name: Update lockfiles

permissions: # (1)!
  contents: write
  pull-requests: write

on:
  workflow_dispatch:
  schedule:
    - cron: 0 5 1 * * # (2)!

jobs:
  pixi-update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up pixi
        uses: prefix-dev/setup-pixi@v0.8.3
        with:
          run-install: false
      - name: Update lockfiles
        run: |
          set -o pipefail
          pixi update --json | pixi exec pixi-diff-to-markdown >> diff.md
      - name: Create pull request
        uses: peter-evans/create-pull-request@v7
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: Update pixi lockfile
          title: Update pixi lockfile
          body-path: diff.md
          branch: update-pixi
          base: main
          labels: pixi
          delete-branch: true
          add-paths: pixi.lock
```

1.  `peter-evans/create-pull-request` 所需
2.  在每月 1 日 05:00 运行

为了让此 workflow 正常工作，你需要将"Allow GitHub Actions to create and approve pull requests"设置为 true（在仓库设置的"Actions"->"General"中）。

!!! tip

    如果你没有任何 `pypi-dependencies`，你可以使用 `pixi update --json --no-install` 来加速 diff 生成。

![Allow GitHub Actions PRs](../../assets/allow-github-actions-prs-light.png#only-light)
![Allow GitHub Actions PRs](../../assets/allow-github-actions-prs-dark.png#only-dark)

## 在自动化的 PR 中触发 CI

为了防止意外的递归 GitHub Workflow 运行，GitHub 决定在使用默认 `GITHUB_TOKEN` 的自动化 PR 上不触发任何 workflow。
有几种方法可以解决这个限制。你可以在 `peter-evans/create-pull-request` 中找到出色的文档，参见[此处](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#triggering-further-workflow-runs)。

## 自定义摘要

你可以通过使用 `pixi-diff-to-markdown` 的命令行参数或在 `pixi.toml` 中的 `[tool.pixi-diff-to-markdown]` 下指定配置来自定义摘要。请参阅 [pixi-diff-to-markdown 文档](https://github.com/pavelzw/pixi-diff-to-markdown) 或运行 `pixi-diff-to-markdown --help` 获取更多信息。

## 使用可重用的 workflow

如果你想在组织中的多个仓库中使用相同的 workflow，你可以创建一个可重用的 workflow。
你可以在 [GitHub 文档](https://docs.github.com/en/actions/using-workflows/reusing-workflows) 中找到更多信息。
