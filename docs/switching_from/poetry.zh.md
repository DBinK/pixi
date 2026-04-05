# 从 Poetry 切换到 Pixi

欢迎阅读本指南，帮助你从 `poetry` 过渡到 `pixi`。
本文档比较了这些工具之间的关键命令和概念，突出了 `pixi` 独特的环境和包管理方法。
使用 `pixi`，你将体验到类似于 `poetry` 的基于 workspace 的工作流程，同时包含 `conda` 生态系统，并允许轻松分享你的工作。

## 为什么选择 Pixi？

Poetry 可能是 Python 生态系统中最接近 Pixi 的工具，就 workspace 管理而言。
在 PyPI 生态系统之上，`pixi` 增加了 conda 生态系统的力量，允许更灵活和强大的环境管理。

## 快速了解差异

| 任务                       | Poetry                                                            | Pixi                                                                                                                                              |
|----------------------------|-------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| 创建环境    | `poetry new myenv`                                                | `pixi init myenv`                                                                                                                                 |
| 运行任务             | `poetry run which python`                                         | `pixi run which python` `pixi` 使用内置的跨平台 shell 运行，而 poetry 使用你的 shell。                                         |
| 安装包       | `poetry add numpy`                                                | `pixi add numpy` 添加 conda 变体。 `pixi add --pypi numpy` 添加 PyPI 变体。                                            |
| 卸载包     | `poetry remove numpy`                                             | `pixi remove numpy` 移除 conda 变体。 `pixi remove --pypi numpy` 移除 PyPI 变体。                                               |
| 构建包         | `poetry build`                                                    | 我们尚未实现包构建和发布                                                                                            |
| 发布包       | `poetry publish`                                                  | 我们尚未实现包构建和发布                                                                                            |
| 读取 pyproject.toml | `[tool.poetry]`                                                   | `[tool.pixi]`                                                                                                                                     |
| 定义依赖      | `[tool.poetry.dependencies]`                                      | `[tool.pixi.dependencies]` 用于 conda，`[tool.pixi.pypi-dependencies]` 或 `[project.dependencies]` 用于 PyPI 依赖                           |
| 依赖定义      | - `numpy = "^1.2.3"`<br/>- `numpy = "~1.2.3"`<br/>- `numpy = "*"` | - `numpy = ">=1.2.3 <2.0.0"`<br/>- `numpy = ">=1.2.3 <1.3.0"`<br/>- `numpy = "*"`                                                                 |
| 锁文件                  | `poetry.lock`                                                     | `pixi.lock`                                                                                                                                       |
| 环境目录       | `~/.cache/pypoetry/virtualenvs/myenv`                             | `./.pixi` 默认为 workspace 目录，使用 [`detached-environments`](../reference/pixi_configuration.md#detached-environments) 可以移动它 |

## 在我的 workspace 中同时支持 `poetry` 和 `pixi`

你可以允许用户在同一个 workspace 中使用 `poetry` 和 `pixi`，它们不会触及彼此的配置或系统部分。
最好复制依赖项，基本上将 `tool.poetry.dependencies` 的精确副本复制到 `tool.pixi.pypi-dependencies`。
确保 `python` 只在 `tool.pixi.dependencies` 中定义，而不是在 `tool.pixi.pypi-dependencies` 中。

!!! Danger "混合 `pixi` 和 `poetry`"
    可以在 `pixi` 环境中使用 `poetry`，但不建议这样做。
    Pixi 以不同于 `poetry` 的方式支持 PyPI 依赖，混合它们可能导致意外行为。
    因为一次只能使用一个包管理器，最好坚持使用一个。

    如果在 Pixi workspace 之上使用 poetry，你将始终需要在 pixi 环境之后安装 `poetry` 环境。
    让 `pixi` 处理 `python` 和 `poetry` 的安装。