你可以使用 prefix.dev、私有 quetz 实例或 anaconda.org 等服务器对 Pixi 进行认证。不同的服务器使用不同的认证方法。在本文档中，我们详细介绍如何对不同服务器进行认证，以及认证信息的存储位置。

```shell
Usage: pixi auth login [OPTIONS] <HOST>

Arguments:
  <HOST>  要认证的主机（例如 repo.prefix.dev）

Options:
      --token <TOKEN>                                要使用的令牌（用于 prefix.dev 认证）
      --username <USERNAME>                          要使用的用户名（用于基本 HTTP 认证）
      --password <PASSWORD>                          要使用的密码（用于基本 HTTP 认证）
      --conda-token <CONDA_TOKEN>                    要在 anaconda.org / quetz 认证中使用的令牌
      --s3-access-key-id <S3_ACCESS_KEY_ID>          S3 访问密钥 ID
      --s3-secret-access-key <S3_SECRET_ACCESS_KEY>  S3 秘密访问密钥
      --s3-session-token <S3_SESSION_TOKEN>          S3 会话令牌
   -v, --verbose...                                   增加日志详细程度
   -q, --quiet...                                     减少日志详细程度
       --color <COLOR>                                日志是否需要着色 [env: PIXI_COLOR=] [default: auto] [possible values: always, never, auto]
       --no-progress                                  隐藏所有进度条，当 stderr 不是终端时总是开启 [env: PIXI_NO_PROGRESS=]
   -h, --help                                         打印帮助
```

不同的选项是"token"、"conda-token"和"username + password"。

token 变体实现标准的"Bearer Token"认证，如 prefix.dev 平台所使用的那样。Bearer Token 作为附加头随每个请求一起发送，形式为 `Authentication: Bearer <TOKEN>`。

conda-token 选项用于 anaconda.org，也可用于 quetz 服务器。使用此选项，令牌作为 URL 的一部分发送，遵循以下方案：`conda.anaconda.org/t/<TOKEN>/conda-forge/linux-64/...`。

最后一个选项 username 和 password 用于"基本 HTTP 认证"。这相当于添加 `http://user:password@myserver.com/...`。这种认证方法可以通过反向 NGinx 或 Apache 服务器相当容易地配置，因此常用于自托管系统。

## 示例

登录 prefix.dev：

```shell
pixi auth login prefix.dev --token pfx_jj8WDzvnuTHEGdAhwRZMC1Ag8gSto8
```

登录 anaconda.org：

```shell
pixi auth login anaconda.org --conda-token xy-72b914cc-c105-4ec7-a969-ab21d23480ed
```

登录基本 HTTP 安全服务器：

```shell
pixi auth login myserver.com --username user --password password
```

登录 S3 存储桶：

```shell
pixi auth login s3://my-bucket --s3-access-key-id <access-key-id> --s3-secret-access-key <secret-access-key>
# 如果你的密钥使用会话令牌，你也可以使用：
pixi auth login s3://my-bucket --s3-access-key-id <access-key-id> --s3-secret-access-key <secret-access-key> --s3-session-token <session-token>
```

!!!note
    S3 认证也通过典型的 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY` 环境变量得到支持，详见 [S3 部分](s3.md)。

## Pixi 在哪里存储认证信息？

认证信息的存储位置取决于系统。默认情况下，Pixi 尝试使用密钥链在你的机器上安全地存储这些敏感信息。

在 Windows 上，凭据存储在"凭据管理器"中。搜索 `rattler`（Pixi 使用的底层库），你应该能找到 Pixi（或基于 rattler 的其他程序）存储的任何凭据。

在 macOS 上，密码存储在密钥链中。你可以使用 macOS 预装的"Keychain Access"程序访问密码。搜索 `rattler`（Pixi 使用的底层库），你应该能找到 Pixi（或基于 rattler 的其他程序）存储的任何凭据。

在 Linux 上，可以使用 `GNOME Keyring`（或 Keyring）访问由 `libsecret` 安全存储的凭据。搜索 `rattler` 应该会列出 Pixi 和其他基于 rattler 的程序存储的所有凭据。

## 回退存储

如果你在没有任何上述密钥链的服务器上运行，Pixi 会回退到将凭据存储在**不安全的** JSON 文件中。
该 JSON 文件位于 `~/.rattler/credentials.json` 并包含凭据。

## 覆盖认证存储

你可以使用 `RATTLER_AUTH_FILE` 环境变量覆盖凭据文件的默认位置。
设置此环境变量后，它是 pixi 使用的唯一认证数据来源。

例如：

```bash
export RATTLER_AUTH_FILE=$HOME/credentials.json
# 你也可以在命令行指定文件
pixi global install --auth-file $HOME/credentials.json ...
```

!!!note
    `RATTLER_AUTH_FILE` 的优先级高于 CLI 参数。

JSON 应遵循以下格式：

```json
{
    "*.prefix.dev": {
        "BearerToken": "your_token"
    },
    "otherhost.com": {
        "BasicHTTP": {
            "username": "your_username",
            "password": "your_password"
        }
    },
    "conda.anaconda.org": {
        "CondaToken": "your_token"
    },
    "s3://my-bucket": {
        "S3Credentials": {
            "access_key_id": "my-access-key-id",
            "secret_access_key": "my-secret-access-key",
            "session_token": null
        }
    }
}
```

注意：如果你在主机中使用通配符，任何子域都将匹配（例如 `*.prefix.dev` 也匹配 `repo.prefix.dev`）。

最后，你可以在[全局配置文件](./../reference/pixi_configuration.md)中设置认证覆盖文件。

## PyPI 认证

目前，我们支持以下 PyPI 认证方法：

1. [keyring](https://pypi.org/project/keyring/) 认证。
2. `.netrc` 文件认证。

我们希望在将来添加更多方法，所以如果你有希望看到的特定方法，请告诉我们。

### keyring 认证

目前，Pixi 支持通过 Python [keyring](https://pypi.org/project/keyring/) 库进行 uv 方式的认证。

#### 安装 keyring

你可以使用 `pixi global install` 安装 keyring：

=== "基本认证"
    ```shell
    pixi global install keyring
    ```
=== "Google Artifact Registry"
    ```shell
    pixi global install keyring --with keyrings.google-artifactregistry-auth
    ```
=== "Azure DevOps Artifacts"
    ```shell
    pixi global install keyring --with keyrings.artifacts
    ```
=== "AWS CodeArtifact"
    ```shell
    pixi global install keyring --with keyrings.codeartifact
    ```

对于其他注册表，你需要调整这些说明以添加正确的 keyring 后端。

#### 配置工作区使用 keyring

=== "基本认证"
    使用 keyring 存储你的凭据，例如：

    ```shell
    keyring set https://my-index/simple your_username
    # 提示符将出现以输入你的密码
    ```

    将以下配置添加到你的 Pixi manifest，确保在注册表 URL 中包含 `your_username@`：

    ```toml
    [pypi-options]
    index-url = "https://your_username@custom-registry.com/simple"
    ```

=== "Google Artifact Registry"
    确保已登录后（例如运行 `gcloud auth login`），将以下配置添加到你的 Pixi manifest：

    ```toml
    [pypi-options]
    extra-index-urls = ["https://oauth2accesstoken@<location>-python.pkg.dev/<project>/<repository>/simple"]
    ```

    !!!Note
        要更轻松地找到此 URL，你可以使用 `gcloud` 命令：

        ```shell
        gcloud artifacts print-settings python --project=<project> --repository=<repository> --location=<location>
        ```

=== "Azure DevOps Artifacts"
    遵循 [`keyrings.artifacts` 说明](https://github.com/jslorrma/keyrings.artifacts?tab=readme-ov-file#usage)并确保 keyring 正常工作后，将以下配置添加到你的 Pixi manifest：

    ```toml
    [pypi-options]
    extra-index-urls = ["https://VssSessionToken@pkgs.dev.azure.com/{organization}/{project}/_packaging/{feed}/pypi/simple/"]
    ```

=== "AWS CodeArtifact"
    确保已登录，例如通过 `aws sso login`，并将以下配置添加到你的 Pixi manifest：

    ```toml
    [pypi-options]
    extra-index-urls = ["https://aws@<your-domain>-<your-account>.d.codeartifact.<your-region>.amazonaws.com/pypi/<your-repository>/simple/"]
    ```

#### 安装你的环境

配置你的[全局配置](../reference/pixi_configuration.md#pypi-config)，或使用 `--pypi-keyring-provider` 标志，该标志可以设置为 `subprocess`（激活）或 `disabled`：

```shell
# 从现有 pixi 工作区
pixi install --pypi-keyring-provider subprocess
```

### `.netrc` 文件

`pixi` 允许你通过使用存储在 `.netrc` 文件中的凭据进行认证来安全地访问私有注册表。

- `.netrc` 文件可以存储在你的主目录（Unix 类系统上为 `$HOME/.netrc`）
- 或在 Windows 上存储在用户配置文件目录中（`%HOME%\_netrc`）。
- 你也可以使用 `NETRC` 变量设置不同的位置（例如 `export NETRC=/my/custom/location/.netrc`）。
  例如 `export NETRC=/my/custom/location/.netrc pixi install`

在 `.netrc` 文件中，你按以下方式存储认证详情：

```sh
machine registry-name
login admin
password admin
```

有关更多详细信息，你可以访问 [.netrc 文档](https://www.ibm.com/docs/en/aix/7.2?topic=formats-netrc-file-format-tcpip)。
