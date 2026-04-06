# JFrog Artifactory

JFrog Artifactory 是一个企业级的工件仓库管理器，支持 conda 包。本指南介绍如何配置 pixi 以使用 Artifactory 作为私有 conda 通道。

## 设置 Artifactory

### 1. 创建 conda 仓库

在你的 Artifactory 实例中，创建一个具有"Conda"包类型的仓库。
仓库 URL 格式为：`https://my-org.jfrog.io/artifactory/<repository-name>/`

Artifactory 支持不同的仓库类型：

- **本地仓库**：存储你自己的私有包。
- **远程仓库**：缓存并镜像上游通道（如 conda-forge）。这减少了外部带宽，加快了下载速度，并在上游通道不可用时提供可用性。
- **虚拟仓库**：将多个本地和远程仓库组合在单个 URL 下。除非你有更高级的用例或迁移，否则通常不需要这些。

常见的设置是创建一个镜像 conda-forge 的远程仓库，然后将其与内部包使用的本地仓库组合，使用虚拟仓库。

![Artifactory repository overview](../assets/artifactory-repository-overview.png)

### 2. 生成身份令牌

要通过 Artifactory 进行认证，你需要生成身份令牌：

1. 点击右上角的用户个人资料，选择 **Edit Profile**

    ![Edit Profile menu](../assets/artifactory-edit-profile-menu.png)

2. 在 **Authentication Settings** 下，点击 **Generate an Identity Token**

    ![Edit Profile page](../assets/artifactory-edit-profile.png)

3. 添加描述（例如"pixi"）并点击 **Next**

    ![Generate an Identity Token dialog](../assets/artifactory-generate-an-identity-token.png)

4. 复制生成的 **Reference Token**

    ![Generated identity token](../assets/artifactory-generate-identity-token.png)

## 通过 pixi 认证

使用 `pixi auth login` 命令通过你的 Artifactory 实例进行认证：

```shell
pixi auth login --token <artifactory-token> https://my-org.jfrog.io
```

这会使用系统的凭据管理器安全地存储令牌。更多关于凭据存储的详细信息，请参阅[认证](authentication.md)。

## 配置通道

将你的 Artifactory 通道添加到你的 `pixi.toml`：

```toml
[workspace]
channels = ["https://my-org.jfrog.io/artifactory/channel-1", "conda-forge"]
```

!!!note "严格的通道优先级"
    Pixi 使用严格的通道优先级。包总是从第一个包含它的通道解析。
    在上面的例子中，如果一个包同时存在于你的 Artifactory 通道和 conda-forge 中，将始终使用来自 Artifactory 的版本。

    这对于以下情况很有用：

    - 使用内部构建覆盖特定包
    - 确保组织内包版本的一致性
    - 使用私有通道中不可用的私有包

    有关通道优先级工作原理的更多详细信息，请参阅[通道逻辑](../advanced/channel_logic.md)。

## 示例配置

以下是一个使用 Artifactory 并以 conda-forge 作为回退的完整示例：

```toml
[workspace]
name = "my-project"
channels = ["https://my-org.jfrog.io/artifactory/internal-packages", "conda-forge"]
platforms = ["linux-64", "osx-arm64", "win-64"]

[dependencies]
python = ">=3.11"
# 如果 Artifactory 中有，将使用它
my-internal-package = "*"
# 这些将来自 conda-forge（由于通道优先级）
numpy = ">=1.24"
pandas = ">=2.0"
```

### 强制使用特定通道

如果你想确保一个包始终来自特定通道（无论优先级如何），请使用 `channel` 键：

```toml
[dependencies]
# 始终使用 conda-forge 的 numpy，即使 Artifactory 中存在
numpy = { version = ">=1.24", channel = "conda-forge" }
# 始终使用 Artifactory 的 internal-lib
internal-lib = { version = "*", channel = "https://my-org.jfrog.io/artifactory/internal-packages" }
```

这在你想覆盖特定包的默认通道优先级时很有用。

## GitHub Actions 与 OIDC

对于 CI/CD 流水线，你可以使用 OIDC（OpenID Connect）通过 Artifactory 进行认证，而不是将长期存在的令牌存储为密钥。这更安全，因为令牌是短期的且自动轮换。

```yaml
- name: Log in to Artifactory
  uses: jfrog/setup-jfrog-cli@279b1f629f43dd5bc658d8361ac4802a7ef8d2d5 # v4.9.1
  id: artifactory
  env:
    JF_URL: https://my-org.jfrog.io
  with:
    disable-job-summary: true
    oidc-provider-name: ${{ vars.ARTIFACTORY_OIDC_PROVIDER }}
    oidc-audience: ${{ vars.ARTIFACTORY_OIDC_AUDIENCE }}

- name: Set up Pixi
  uses: prefix-dev/setup-pixi@82d477f15f3a381dbcc8adc1206ce643fe110fb7 # v0.9.3
  with:
    auth-host: https://my-org.jfrog.io
    auth-token: ${{ steps.artifactory.outputs.oidc-token }}
```

这需要在你信任 GitHub Actions 的 Artifactory 实例中配置 OIDC 提供商。
有关设置说明，请参阅 JFrog 关于 [OIDC 集成](https://jfrog.com/help/r/jfrog-platform-administration-documentation/configure-an-oidc-integration)的文档。
