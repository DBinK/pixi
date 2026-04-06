[Visual Studio Code](https://code.visualstudio.com/) 是一个流行的编辑器，可以通过安装适当的扩展来支持大多数编程语言。

## Python 扩展

首先，从 [marketplace](https://marketplace.visualstudio.com/items?itemName=ms-python.python) 安装 Python 扩展。通常，一旦你打开 Python 文件，扩展会自动检测并选择 Pixi 默认环境。如果它没有自动选择，或者你想选择不同的环境，你可以打开环境选择器来选择你选择的环境。

![VSCode Python Environment Selector](../../assets/vscode-python-env-selector.png)

## Direnv 扩展

Direnv 提供了一种与语言无关的方式在 Pixi 环境中运行 VSCode。首先，从 [marketplace](https://marketplace.visualstudio.com/items?itemName=mkhl.direnv) 安装 Direnv 扩展。然后按照我们的 [Direnv 文档页面](../third_party/direnv.md) 中的说明进行操作。

## Devcontainer 扩展

[VSCode Devcontainers](https://code.visualstudio.com/docs/devcontainers/containers) 是一种在具有一致环境的 workspace 上开发的流行工具。它们还用于 [GitHub Codespaces](https://github.com/features/codespaces)，这使得在不需要在本地机器上安装任何东西的情况下在 workspace 上开发成为一种很棒的方式。

要在 devcontainer 中使用 pixi，请按以下步骤操作：

在工作区根目录中创建一个新目录 `.devcontainer`。
然后在 `.devcontainer` 目录中创建以下两个文件：

```dockerfile title=".devcontainer/Dockerfile"
FROM mcr.microsoft.com/devcontainers/base:jammy

ARG PIXI_VERSION=v0.66.0

RUN curl -L -o /usr/local/bin/pixi -fsSL --compressed "https://github.com/prefix-dev/pixi/releases/download/${PIXI_VERSION}/pixi-$(uname -m)-unknown-linux-musl" \
    && chmod +x /usr/local/bin/pixi \
    && pixi info

# 设置一些用户和 workdir 设置以与 vscode 配合使用
USER vscode
WORKDIR /home/vscode

RUN echo 'eval "$(pixi completion -s bash)"' >> /home/vscode/.bashrc
```

```json title=".devcontainer/devcontainer.json"
{
    "name": "my-workspace",
    "build": {
      "dockerfile": "Dockerfile",
      "context": "..",
    },
    "customizations": {
      "vscode": {
        "settings": {},
        "extensions": ["ms-python.python", "charliermarsh.ruff", "GitHub.copilot"]
      }
    },
    "features": {
      "ghcr.io/devcontainers/features/docker-in-docker:2": {}
    },
    "mounts": ["source=${localWorkspaceFolderBasename}-pixi,target=${containerWorkspaceFolder}/.pixi,type=volume"],
    "postCreateCommand": "sudo chown vscode .pixi && pixi install"
}
```

!!!tip "将 `.pixi` 放在挂载中"
    在上面的示例中，我们将 `.pixi` 目录挂载到卷中。
    这是必需的，因为 `.pixi` 目录不应该放在大小写不敏感的文件系统上（macOS 和 Windows 上的默认设置），而应该放在自己的卷中。
    有些 conda 包（例如 [ncurses-feedstock#73](https://github.com/conda-forge/ncurses-feedstock/issues/73)）包含大小写不同的文件，这会导致大小写不敏感的文件系统出错。

## 密钥

如果你想通过私有 conda 通道进行认证，你可以将密钥添加到你的 devcontainer。

```json title=".devcontainer/devcontainer.json"
{
    "build": "Dockerfile",
    "context": "..",
    "options": [
        "--secret",
        "id=prefix_dev_token,env=PREFIX_DEV_TOKEN",
    ],
    // ...
}
```

```dockerfile title=".devcontainer/Dockerfile"
# ...
RUN --mount=type=secret,id=prefix_dev_token,uid=1000 \
    test -s /run/secrets/prefix_dev_token \
    && pixi auth login --token "$(cat /run/secrets/prefix_dev_token)" https://repo.prefix.dev
```

这些密钥需要作为环境变量在本地启动 devcontainer 时存在，或者在 [GitHub Codespaces 设置](https://github.com/settings/codespaces) 中的 `Secrets` 下存在。
