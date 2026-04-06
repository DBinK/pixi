# Prefix.dev

Prefix.dev 为 Conda 包提供了快速、现代化的包托管解决方案。

## 在 prefix.dev 上创建自己的通道

首先，在 prefix.dev 上注册账户，可以使用 GitHub、Google 或邮箱地址。
通过导航到 `Channels` 并点击 "New Channel" 来创建通道。

![Create new channel](../assets/prefix_create_channel.png)

填写通道创建表单：选择**名称**、描述以及通道是公开还是私有。公开通道无需认证即可访问。你也可以使用 GitHub 头像 URL 作为通道 Logo（例如 `https://github.com/myaccount.png`）。

![Channel creation form](../assets/prefix_channel_creation.png)

你的通道已创建！现在你可以开始上传包或添加成员。

## 添加更多通道成员

要管理成员，导航到通道设置并点击侧边栏中的 **Members**。

![Members overview](../assets/prefix_members.png)

要邀请新成员，点击 **Invite member**，搜索用户名并选择角色（例如 Contributor 或 Viewer），然后点击 **Add**。注意：一个通道只能有**一个**所有者，但你也可以在通道设置中转移通道所有权。

![Invite a member](../assets/prefix_edit_members.png)

## 上传包

你可以直接通过 Web 界面上传包。导航到通道设置侧边栏中的 **Upload Package**。将 `.conda` 或 `.tar.bz2` 文件（最大 1 GB）拖放到上传区域，或点击从文件系统选择文件。

![Upload a package](../assets/prefix_upload_package.png)

或者，你可以使用 `pixi` CLI 构建和发布包。要在本地执行此操作，需要为你的用户账户生成 API 密钥（User -> Settings -> Api Keys）。

最简单的方式是使用 `pixi publish`，它可以一步完成包的构建和上传：

```bash
# 一步完成构建和发布
pixi publish --to https://prefix.dev/<channel-name>
```

你也可以分步构建和上传以获得更多控制：

```bash
# 先构建，然后上传产物
pixi build --output-dir ./output
pixi upload prefix --channel <channel-name> ./output/my-package-1.0.0-h123_0.conda
```

要进行认证，使用 `pixi auth login` 或传递 API 密钥：

```bash
# 将凭据存储到密钥链
pixi auth login --token $PREFIX_API_KEY https://prefix.dev

# 或在上传时直接传递 API 密钥
pixi upload prefix --channel <channel-name> <package-file> --api-key $PREFIX_API_KEY
```

## 在 pixi 中使用通道

一旦你的通道设置完成并且有包可用，你可以在 `pixi.toml` 中使用它：

```toml
[workspace]
channels = ["https://prefix.dev/<channel-name>"]
```

对于私有通道，需要先进行认证：

```bash
pixi auth login --token <your-token> https://prefix.dev
```

## 删除或转移通道

在通道设置的 **Delete or Transfer** 中，你可以将通道的所有权转移给其他用户，或永久删除通道及其所有包。

![Danger zone](../assets/prefix_delete_channel.png)

!!! warning
    删除通道是不可逆的，会删除所有与之关联的包。请谨慎操作。

## 受信任发布

受信任发布允许 CI/CD 流水线将包上传到你的通道，而无需使用长期存在的 API 令牌。相反，它使用 OIDC（OpenID Connect）在你的 CI 提供商和 prefix.dev 之间建立信任。因为凭据是短期的且自动限定到特定工作流，所以这更安全。

Prefix.dev 支持来自 **GitHub Actions**、**GitLab CI/CD** 和 **Google Cloud** 的受信任发布。

### 设置受信任发布者

导航到通道设置侧边栏中的 **Trusted Publishers** 并填写必填字段：

- **GitHub Username or Organization Name** — 仓库的所有者
- **Repository Name** — 将发布包的仓库
- **Name of the Workflow File** — 触发上传的工作流文件（例如 `release-workflow.yml`）
- **Environment Name**（可选）— 限制发布到特定的 GitHub 环境（例如 `production`）

![Trusted Publishing](../assets/prefix_trusted_publishing.png)

### 在 GitHub Actions 中使用受信任发布

配置后，你的 GitHub Actions 工作流可以在没有存储密钥的情况下上传包：

```yaml
jobs:
  build-and-upload:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # OIDC 令牌必填
    steps:
      - uses: actions/checkout@v4

      - uses: prefix-dev/setup-pixi@v0.8.0

      # 一步完成构建和发布 — 无需存储密钥！
      - run: pixi publish --to https://prefix.dev/<channel-name>

      # 或使用 sigstore 证明以确保供应链安全：
      # - run: pixi publish --to https://prefix.dev/<channel-name> --generate-attestation
```

配置受信任发布后，pixi 会自动处理与 prefix.dev 的 OIDC 令牌交换 — 无需存储 API 密钥。
