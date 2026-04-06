
# Pixi 本身的配置

除了[工作区特定配置](../reference/pixi_manifest.md)之外，Pixi 还支持不需要工作区工作但本地于机器的配置选项。配置按以下顺序加载：

=== "Linux"

    | **优先级** | **位置**                                                           | **注释**                                          |
    |----------|-------------------------------------------------------------------|---------------------------------------------------|
    | 7        | 命令行参数（`--tls-no-verify`、`--change-ps1=false` 等）            | 通过命令行参数进行配置                             |
    | 6        | `your_project/.pixi/config.toml`                                  | 项目特定配置                                      |
    | 5        | `$PIXI_HOME/config.toml`                                          | `PIXI_HOME` 中的全局配置。                         |
    | 4        | `$HOME/.pixi/config.toml`                                         | 用户主目录中的全局配置。                           |
    | 3        | `$XDG_CONFIG_HOME/pixi/config.toml`                               | XDG 兼容的用户特定配置                            |
    | 2        | `$HOME/.config/pixi/config.toml`                                  | 用户特定配置                                      |
    | 1        | `/etc/pixi/config.toml`                                           | 系统范围配置                                      |

=== "macOS"

    | **优先级** | **位置**                                                           | **注释**                                          |
    |----------|-------------------------------------------------------------------|---------------------------------------------------|
    | 7        | 命令行参数（`--tls-no-verify`、`--change-ps1=false` 等）            | 通过命令行参数进行配置                             |
    | 6        | `your_project/.pixi/config.toml`                                  | 项目特定配置                                      |
    | 5        | `$PIXI_HOME/config.toml`                                          | `PIXI_HOME` 中的全局配置。                         |
    | 4        | `$HOME/.pixi/config.toml`                                         | 用户主目录中的全局配置。                           |
    | 3        | `$HOME/Library/Application Support/pixi/config.toml`              | 用户特定配置                                      |
    | 2        | `$XDG_CONFIG_HOME/pixi/config.toml`                               | XDG 兼容的用户特定配置                            |
    | 1        | `/etc/pixi/config.toml`                                           | 系统范围配置                                      |

=== "Windows"

    | **优先级** | **位置**                                                           | **注释**                                          |
    |----------|-------------------------------------------------------------------|---------------------------------------------------|
    | 6        | 命令行参数（`--tls-no-verify`、`--change-ps1=false` 等）            | 通过命令行参数进行配置                             |
    | 5        | `your_project\.pixi\config.toml`                                  | 项目特定配置                                      |
    | 4        | `%PIXI_HOME%\config.toml`                                         | `PIXI_HOME` 中的全局配置。                         |
    | 3        | `%USERPROFILE%\.pixi\config.toml`                                 | 用户主目录中的全局配置。                           |
    | 2        | `%APPDATA%\pixi\config.toml`                                       | 用户特定配置                                      |
    | 1        | `C:\ProgramData\pixi\config.toml`                                 | 系统范围配置                                      |

!!! note
    优先级最高者胜出。如果在更高优先级位置找到配置文件，则来自较低优先级位置读取的值将被覆盖。

!!! note
    要查找 `pixi` 查找配置文件的位置，请运行
    `pixi info -vvv`。

## 配置选项

??? info "配置中的命名约定"
    在 Pixi `0.20.1` 及更早版本中，全局配置选项使用 `snake_case`，我们已将其更改为 `kebab-case` 以与配置的其余部分保持一致。
    为了向后兼容，以下配置选项仍可以用 `snake_case` 编写：

    - `default_channels`
    - `change_ps1`
    - `tls_no_verify`
    - `authentication_override_file`
    - `mirrors` 及其子选项
    - `repodata_config` 及其子选项

以下参考描述了所有可用的配置选项。

### `default-channels`

运行 `pixi init` 或 `pixi global install` 时选择的默认通道。默认为仅 conda-forge。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:default-channels"
```

!!! note
    `default-channels` 仅在初始化新工作区时使用。初始化后，`channels` 来自工作区清单。

### `shell`

- `change-ps1`：设置为 `false` 时，shell 提示符中的 `(pixi)` 前缀将被移除。
  这适用于 `pixi shell` 子命令。
  你可以通过 CLI 使用 `--change-ps1` 覆盖此选项。
- `force-activate`：设置为 `true` 时，环境的重新激活将始终发生。
  这与 [`experimental`](#experimental) 功能 `use-environment-activation-cache` 结合使用。
- `source-completion-scripts`：设置为 `false` 时，Pixi 在进入 shell 时不会 source 环境的自动补全脚本。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:shell"
```

### `tls-no-verify`

设置为 true 时，所有网络连接（包括 conda 通道和 PyPI 注册表）都将禁用 TLS 证书验证。
你可以使用 CLI 中的 `--tls-no-verify` 覆盖此选项。

!!! warning

    这是安全风险，应仅用于测试目的或内部网络。

这是一个影响所有连接的全局设置。
如果你只需要为特定 PyPI 主机绕过 TLS 验证，考虑使用 [`pypi-config.allow-insecure-host`](#pypi-config) 以获得更细粒度的控制。

!!! note "实现细节和限制"

    对于 PyPI 操作，由于 uv 不支持全局 TLS 验证禁用标志，pixi 在启用 `tls-no-verify` 时会自动将所有配置的 PyPI 索引主机添加到受信任主机列表。
    对于主要 PyPI 索引，`files.pythonhosted.org`（下载主机）也会被自动添加。
    **重要限制：** 如果你的自定义 PyPI 索引将包下载重定向到不同的主机（例如，单独的 CDN 或工件服务器），即使设置了 `tls-no-verify`，该下载主机也不会被自动信任。
    在这些情况下，你必须手动将下载主机添加到 [`pypi-config.allow-insecure-host`](#pypi-config)。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:tls-no-verify"
```

### `tls-root-certs`

控制用于 HTTPS 连接的 TLS 根证书。这会影响 conda 通道和 PyPI 注册表。

可用选项：

- `webpki`（默认）：使用捆绑的 Mozilla 根证书。这是最可移植的选项。
- `native`：使用系统证书存储。对于具有自定义 CA 证书的企业环境是必需的。
- `all`：同时使用捆绑的 Mozilla 证书和系统证书存储。

你可以使用 CLI 中的 `--tls-root-certs` 覆盖此选项。

!!! note "构建依赖的行为"

    此设置仅对 `rustls-tls` 构建有效（来自 GitHub releases 的独立 pixi 二进制文件）。
    对于 `native-tls` 构建（conda-forge 包），使用系统 TLS 库，该库始终使用系统证书。
    在这种情况下，设置被接受但无效。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:tls-root-certs"
```

### `authentication-override-file`

覆盖从中加载认证信息的位置。通常，我们尝试使用密钥链加载认证数据，并且仅将 JSON
文件作为回退。此选项允许你强制使用 JSON 文件。
在认证部分阅读更多内容。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:authentication-override-file"
```

### `detached-environments`

Pixi 存储工作区环境的目录，这些环境通常放在工作区根目录的 `.pixi/envs` 文件夹中。
它不影响为 `pixi global` 构建的环境。
为 `pixi global` 安装创建的环境位置可以使用 `PIXI_HOME`
环境变量控制。

!!! warning

    我们建议不要使用此选项，因为为工作区创建的任何环境不再放在与工作区相同的文件夹中。
    这在工作区与其环境之间造成了脱节，删除工作区时需要手动清理环境。

    然而，在某些情况下，此选项仍然非常有用，例如：

    - 强制安装到特定文件系统/驱动器。
    - 本地安装环境但将工作区保留在网络驱动器上。
    - 让系统管理员更好地控制系统上的所有环境。

此字段可以由两种类型的输入组成。

- 布尔值，`true` 或 `false`，将分别启用或禁用该功能。（不是 `"true"` 或 `"false"`，
  这被读取为 `false`）
- 字符串值，将是存储环境的目录的绝对路径。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:detached-environments"
```

或者：

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/detached_environments_path_config.toml:detached-environments-path"
```

当此选项为 `true` 时，环境将存储在[缓存目录](../workspace/environment.md#caching-packages)中。
当你指定自定义路径时，环境将存储在该目录中。

生成的目录结构将如下所示：

```shell
/opt/pixi/envs
├── pixi-6837172896226367631
│   └── envs
└── NAME_OF_PROJECT-HASH_OF_ORIGINAL_PATH
    ├── envs # 可运行的环境
    └── solve-group-envs # 如果有 solve groups

```

### `pinning-strategy`

运行 `pixi add` 时用于固定依赖项的策略。默认为 `semver`，但你可以设置以下选项：

- `no-pin`：不固定，导致未约束的依赖项。`*`
- `semver`：固定到满足 semver 约束的最新版本。结果对于大多数版本固定到 major，对于 `v0` 版本固定到 minor。
- `exact-version`：固定到精确版本，`1.2.3` -> `==1.2.3`。
- `major`：固定到 major 版本，`1.2.3` -> `>=1.2.3, <2`。
- `minor`：固定到 minor 版本，`1.2.3` -> `>=1.2.3, <1.3`。
- `latest-up`：固定到最新版本，`1.2.3` -> `>=1.2.3`。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:pinning-strategy"
```

### `mirrors`

conda 通道镜像的配置，更多信息在[下面](#mirror-configuration)。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:mirrors"
```

### `proxy-config`

`pixi` 尊重优先级最高的代理环境，如 `https_proxy`。
我们也可以在 `pixi` 配置文件的 `proxy-config` 表中设置代理，这会影响所有 pixi
网络操作，如解析、自我更新、下载等。
现在 `proxy-config` 表支持以下选项：

- `https` 和 `http`：`https://` 或 `http://` URL 的代理，工作方式类似于 `https_proxy` 和 `http_proxy` 环境。
- `non-proxy-hosts`：应该绕过代理的域名列表。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:proxy-config"
```

### `repodata-config`

repodata 获取的配置。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:repodata-config"
```

可以通过在配置中指定通道前缀来覆盖每个通道的上述设置。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:prefix-repodata-config"
```

### `pypi-config`

设置使用 PyPI 注册表的一些默认值。你可以使用以下配置选项：

- `index-url`：用于 PyPI 包的默认索引 URL。这将在 `pixi init` 时添加到清单文件。
- `extra-index-urls`：用于 PyPI 包的附加 URL 列表。这将在 `pixi init` 时添加到清单文件。
- `keyring-provider`：允许使用 [keyring](https://pypi.org/project/keyring/) Python 包来存储和检索凭据。
- `allow-insecure-host`：应禁用 TLS 证书验证的主机名列表（无协议或端口），在访问 PyPI 注册表时。这在处理使用自签名证书的内部 PyPI 镜像时很有用。要全局禁用所有连接的 TLS 验证，请改用 [`tls-no-verify`](#tls-no-verify)。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:pypi-config"
```

!!! Note "`index-url` 和 `extra-index-urls` 不是全局的"
    与 pip 不同，这些设置（`keyring-provider` 除外）只会修改 `pixi.toml`/`pyproject.toml`
    文件，并且在清单中不存在时不会全局解释。
    这是因为我们希望尽可能保持清单文件的完整性和可重现性。

### `s3-options`

S3 认证的配置。这将导致 Pixi 不使用 AWS 的默认凭据，而是使用
Pixi 认证存储中的凭据，详见 [S3 部分](../deployment/s3.md) 了解更多信息。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:s3-options"
```

### `concurrency`

配置多个设置以限制或扩展 pixi 的并发。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:concurrency"
```

通过 CLI 设置它们：

```shell
pixi config set concurrency.solves 1
pixi config set concurrency.downloads 12
```

### `run-post-link-scripts`

配置 pixi 是否应执行 `post-link` 和 `pre-unlink` 脚本。

某些包包含在包安装后执行的 post-link 脚本（`bat` 或 `sh` 文件）。我们认为
这些脚本是不安全的，因为它们可能包含在用户不知情的情况下在用户机器上执行的任意代码。默认情况下，`run-post-link-scripts` 设置为 `false`，防止执行这些脚本。

但是，你可以通过将值设置为 `insecure`（例如运行
`pixi config set --local run-post-link-scripts insecure`）在全球（或工作区）范围内选择加入。

未来我们计划添加 `sandbox` 模式以在受控环境中执行这些脚本。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:run-post-link-scripts"
```

### `tool-platform`

定义安装"工具"（如构建后端）时使用的平台。默认情况下 pixi 使用当前
系统的平台，但你可以覆盖它使用不同的平台。如果你正在运行 pixi 的架构对某些构建后端支持较少，这可能很有用。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:tool-platform"
```

!!! Note "虚拟包"
    工具平台的虚拟包是从当前系统检测到的。如果工具平台与当前系统操作系统不同，将不会使用虚拟包。

## 实验性

这允许用户设置尚未稳定的特定实验性功能。

请在发现激活的功能有问题时添加 `experimental` 标志提交 GitHub 问题。

### 缓存环境激活

使用以下命令从配置中开启此功能：

```shell
# 对于所有你的工作区
pixi config set experimental.use-environment-activation-cache true --global

# 对于特定工作区
pixi config set experimental.use-environment-activation-cache true --local
```

这将在工作区根目录的 `.pixi/activation-env-v0` 文件夹中缓存环境激活。
它将为每个激活的环境创建一个 json 文件，并在将来使用它来激活环境。

```bash
> tree .pixi/activation-env-v0/
.pixi/activation-env-v0/
├── activation_default.json
└── activation_lint.json

> cat  .pixi/activation-env-v0/activation_lint.json
{"hash":"8d8344e0751d377a","environment_variables":{<ENVIRONMENT_VARIABLES_USED_IN_ACTIVATION>}}
```

- `hash` 是 `pixi.lock` 中该环境数据的哈希，加上环境激活的一些重要信息。
  如清单文件中的 `[activation.scripts]` 和 `[activation.env]`。
- `environment_variables` 是激活环境时设置的环境变量。

你可以通过运行以下命令忽略缓存：

```
pixi run/shell/shell-hook --force-activate
```

使用以下命令设置配置：

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/main_config.toml:experimental"
```

!!! note "为什么这是实验性的？"
    此功能是实验性的，因为缓存失效非常棘手，
    我们不想打扰不受激活时间影响的用户。

## 镜像配置

你可以为 conda 通道配置镜像。我们希望镜像是原始通道的精确副本。实现将在配置文件的 `mirrors` 部分中查找镜像键（URL）并用镜像 URL 替换原始 URL。

要包含原始 URL，你必须在镜像列表中重复它。

镜像基于列表的优先级排序。我们将尝试从列表中的第一个镜像获取 repodata（最重要的文件）。
repodata 包含所有单个包的 SHA256 哈希，因此从可信来源获取此文件很重要。

你也可以为整个"主机"指定镜像，例如。

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/mirror_prefix_config.toml:mirrors)
```

这将把所有对 anaconda.org 上通道的请求转发到 prefix.dev。当前未在 prefix.dev 上镜像的通道将在上面的示例中失败。
你可以通过提供指向自身的更长前缀来覆盖特定通道（例如 conda-forge 的标签通道）的行为。

### OCI 镜像

你也可以在 OCI 注册表上指定镜像。GitHub 容器注册表（ghcr.io）上有一个由 conda-forge 团队维护的公共镜像。你可以这样使用它：

```toml title="config.toml"
--8<-- "docs/source_files/pixi_config_tomls/oci_config.toml:oci-mirrors"
```

GHCR 镜像还包含 `bioconda` 包。你可以在 [Github](https://github.com/orgs/channel-mirrors/packages) 上搜索可用的包。

### PyPi 解析和 PyPi 包下载的镜像

镜像也影响 `uv` 的 PyPi 解析和下载步骤。你可以为主要的 pypi 索引和额外索引设置镜像。但下载 url 的基础部分可能与它们的索引 url 不同。例如，默认的 pypi 索引是 `https://pypi.org/simple`，包下载 url 的形式是 `https://files.pythonhosted.org/packages/<h1>/<h2>/<hash>/<package>.whl`。你需要两个镜像配置条目用于 `https://pypi.org/simple` 和 `https://files.pythonhosted.org/packages`。
