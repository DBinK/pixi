我们创建了 [prefix-dev/setup-pixi](https://github.com/prefix-dev/setup-pixi) 来简化在 CI 中使用 pixi。

## 使用方法

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    pixi-version: v0.66.0
    cache: true
    auth-host: prefix.dev
    auth-token: ${{ secrets.PREFIX_DEV_TOKEN }}
- run: pixi run test
```

!!!warning "固定你的 action 版本"
    由于 pixi 尚不稳定，此 action 的 API 可能会在次版本之间发生变化。
    请将此 action 的版本固定到特定版本（即 `prefix-dev/setup-pixi@v0.9.4`）以避免破坏性更改。
    你可以通过使用 [Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot) 自动更新此 action 的版本。

    将以下内容放入你的 `.github/dependabot.yml` 文件以启用 Dependabot 来管理你的 GitHub Actions：

    ```yaml title=".github/dependabot.yml"
    version: 2
    updates:
      - package-ecosystem: github-actions
        directory: /
        schedule:
          interval: monthly # (1)!
        groups:
          dependencies:
            patterns:
              - "*"
        cooldown:
          default-days: 7
    ```

    1.  或 `daily`、`weekly`

## 功能

有关所有可用的输入参数，请参阅 `setup-pixi` 中的 [`action.yml`](https://github.com/prefix-dev/setup-pixi/blob/main/action.yml) 文件。
最重要的功能在下面介绍。

### 缓存

该 action 支持缓存项目环境和全局 pixi 环境。
默认情况下，如果存在 `pixi.lock` 文件，则启用项目环境缓存。
然后它将使用 `pixi.lock` 文件生成环境的哈希并缓存它。
如果缓存命中，action 将跳过安装并使用缓存的环境。
你可以通过设置 `cache` 输入参数来指定行为。

默认情况下，全局环境缓存是禁用的，可以通过将 `global-cache` 输入设置为 `true` 来启用。
由于全局环境没有 lockfile，缓存将在每个月月底过期，以确保不会变陈旧。

!!!tip "自定义你的缓存键"
    如果你需要自定义你的缓存键，你可以使用 `cache-key` 和 `global-cache-key` 输入参数。
    这些将是缓存键的前缀。完整的缓存键分别是 `<cache-key><conda-arch>-<hash>` 和 `<global-cache-key><conda-arch>-<YYYY-MM>-<hash>`。

!!!tip "只在 `main` 上保存缓存"
    为了不超过[10 GB 缓存大小限制](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy)，你可能需要限制何时保存缓存。
    这可以通过设置 `cache-write` 参数来完成。

    ```yaml
    - uses: prefix-dev/setup-pixi@v0.9.4
      with:
        cache: true
        cache-write: ${{ github.event_name == 'push' && github.ref_name == 'main' }}
    ```

### 多个环境

使用 pixi，你可以为不同的需求创建多个环境。
你也可以通过设置 `environments` 输入参数来指定你想安装哪些环境。
这将安装所有指定的环境并缓存它们。

```toml
[workspace]
name = "my-package"
channels = ["conda-forge"]
platforms = ["linux-64"]

[dependencies]
python = ">=3.11"
pip = "*"
polars = ">=0.14.24,<0.21"

[feature.py311.dependencies]
python = "3.11.*"
[feature.py312.dependencies]
python = "3.12.*"

[environments]
py311 = ["py311"]
py312 = ["py312"]
```

#### 使用矩阵的多个环境

以下示例将在不同的 job 中安装 `py311` 和 `py312` 环境。

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      environment: [py311, py312]
  steps:
  - uses: actions/checkout@v4
  - uses: prefix-dev/setup-pixi@v0.9.4
    with:
      environments: ${{ matrix.environment }}
```

#### 在一个 job 中安装多个环境

以下示例将在 runner 上同时安装 `py311` 和 `py312` 环境。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    environments: >- # (1)!
      py311
      py312
- run: |
    pixi run -e py311 test
    pixi run -e py312 test
```

1.  用空格分隔，等同于

   ```yaml
   environments: py311 py312
   ```

!!!warning "如果你不指定环境时的缓存行为"
    如果你不指定任何环境，即使你使用其他环境，`default` 环境也将被安装和缓存。

### 全局环境

你可以通过设置 `global-environments` 输入参数来指定 `pixi global install` 命令。
这将为每行创建一个环境并安装它们。
这在安装 `pixi install` 正常工作所需的可执行文件时尤其有用。
例如，`keyring` 或 `gcloud` 可执行文件。以下示例展示了如何分别安装两者。
默认情况下，全局环境不会被缓存。你可以通过将 `global-cache` 输入设置为 `true` 来启用缓存。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    global-environments: |
      google-cloud-sdk
      keyring --with keyrings.google-artifactregistry-auth
- run: |
    gcloud --version
    keyring --list-backends
```

### 认证

目前有五种方式与 pixi 进行认证：

- 使用 token
- 使用用户名和密码
- 使用 conda-token
- 使用 S3 密钥对
- 使用 keyring 进行 PyPI 注册表

有关更多信息，请参阅[认证](../../deployment/authentication.md)。

!!!warning "谨慎处理密钥"
    请仅使用 [GitHub 密钥](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) 存储敏感信息。不要将它们存储在你的仓库中。
    当你的敏感信息存储在 GitHub 密钥中时，你可以使用 `${{ secrets.SECRET_NAME }}` 语法访问它。
    这些密钥在日志中始终会被屏蔽。

#### Token

使用 `auth-token` 输入参数指定 token。
这种认证形式（请求头中的 bearer token）主要用于 [prefix.dev](https://prefix.dev)。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    auth-host: prefix.dev
    auth-token: ${{ secrets.PREFIX_DEV_TOKEN }}
```

#### 用户名和密码

使用 `auth-username` 和 `auth-password` 输入参数指定用户名和密码。
这种认证形式（HTTP Basic Auth）用于某些企业环境，例如与 [artifactory](https://jfrog.com/artifactory) 一起使用。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    auth-host: custom-artifactory.com
    auth-username: ${{ secrets.PIXI_USERNAME }}
    auth-password: ${{ secrets.PIXI_PASSWORD }}
```

#### Conda-token

使用 `conda-token` 输入参数指定 conda-token。
这种认证形式（token 编码在 URL 中：`https://我的-quetz-实例.com/t/<token>/get/custom-channel`）用于 [anaconda.org](https://anaconda.org) 或 [quetz 实例](https://github.com/mamba-org/quetz)。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    auth-host: anaconda.org # (1)!
    auth-conda-token: ${{ secrets.CONDA_TOKEN }}
```

1.  或我的-quetz-实例.com

#### S3

使用 `auth-access-key-id` 和 `auth-secret-access-key` 输入参数指定 S3 密钥对。
你也可以使用 `auth-session-token` 输入参数指定会话令牌。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    auth-host: s3://my-s3-bucket
    auth-s3-access-key-id: ${{ secrets.ACCESS_KEY_ID }}
    auth-s3-secret-access-key: ${{ secrets.SECRET_ACCESS_KEY }}
    auth-s3-session-token: ${{ secrets.SESSION_TOKEN }} # (1)!
```

1.  仅当你的密钥使用会话令牌时才需要

有关 S3 认证的更多信息，请参阅 [S3 部分](../../deployment/s3.md)。

#### PyPI keyring provider

你可以指定是否使用 keyring 查找 PyPI 的凭据。

```yml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    pypi-keyring-provider: subprocess # 之一是 'subprocess', 'disabled'
```

### 自定义 shell 包装器

`setup-pixi` 允许你通过使用自定义 shell 包装器 `shell: pixi run bash -e {0}` 在 pixi 环境中运行命令。
如果你想在 pixi 环境中运行命令，但不想为每个命令使用 `pixi run` 命令，这会很有用。

```yaml
- run: | # (1)!
    python --version
    pip install --no-deps -e .
  shell: pixi run bash -e {0}
```

1.  这里的所有内容都将在 pixi 环境中运行

你甚至可以像这样运行 Python 脚本：

```yaml
- run: | # (1)!
    import my_package
    print("Hello world!")
  shell: pixi run python {0}
```

1.  这里的所有内容都将在 pixi 环境中运行

如果你想使用 PowerShell，你还需要指定 `-Command`。

```yaml
- run: | # (1)!
    python --version | Select-String "3.11"
  shell: pixi run pwsh -Command {0} # pwsh 适用于所有平台
```

1.  这里的所有内容都将在 pixi 环境中运行

!!!note "它在后台是如何工作的？"
    在后台，`shell: xyz {0}` 选项通过创建临时脚本文件并使用该脚本文件作为参数调用 `xyz` 来实现。
    此文件没有设置可执行位，因此你不能直接使用 `shell: pixi run {0}`，而是必须使用 `shell: pixi run bash {0}`。
    GitHub 提供了一些具有稍微不同行为的自定义 shell，请参阅文档中的 [`jobs.<job_id>.steps[*].shell`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell)。
    有关 `shell:` 输入在 GitHub Actions 中如何工作的更多信息，请参阅[官方文档](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#custom-shell) 和 [ADR 0277](https://github.com/actions/runner/blob/main/docs/adrs/0277-run-action-shell-options.md)。

#### 使用 `pixi exec` 的一次性 shell 包装器

使用 `pixi exec`，你也可以在临时 pixi 环境中运行一次性命令。

```yaml
- run: | # (1)!
    zstd --version
  shell: pixi exec --spec zstd -- bash -e {0}
```

1.  这里的所有内容都将在临时 pixi 环境中运行

```yaml
- run: | # (1)!
    import ruamel.yaml
    # ...
  shell: pixi exec --spec python=3.11.* --spec ruamel.yaml -- python {0}
```

1.  这里的所有内容都将在临时 pixi 环境中运行

有关 `pixi exec` 的更多信息，请参阅[此处](../../reference/cli/pixi/exec.md)。

### 环境激活

除了使用自定义 shell 包装器，你还可以通过"激活"当前运行的 job 中安装的环境，使所有 pixi 安装的二进制文件对后续步骤可用。
为此，`setup-pixi` 会添加执行 `pixi run` 时设置的所有环境变量到 `$GITHUB_ENV`，同样会将所有路径修改添加到 `$GITHUB_PATH`。
因此，所有已安装的二进制文件都可以在不需要调用 `pixi run` 的情况下访问。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    activate-environment: true
```

如果你正在安装多个环境，你需要指定你希望激活的环境名称。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    environments: >-
      py311
      py312
    activate-environment: py311
```

激活环境可能比使用自定义 shell 包装器更有用，因为它允许非基于 shell 的步骤访问路径上的二进制文件。
但是请注意，此选项会增强你的 job 环境。

### `--frozen` 和 `--locked`

你可以根据 `frozen` 或 `locked` 输入参数指定 `setup-pixi` 应该运行 `pixi install --frozen` 还是 `pixi install --locked`。
有关 `--frozen` 和 `--locked` 标志的更多信息，请参阅[官方文档](../../reference/cli/pixi/install.md#update-options)。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    locked: true
    # 或
    frozen: true
```

如果你不指定任何内容，默认行为是如果存在 `pixi.lock` 文件则运行 `pixi install --locked`，否则运行 `pixi install`。

### 调试

你可以启用两种类型的调试日志。

#### Action 的调试日志

第一个是 action 本身的调试日志。
可以通过在调试模式下重新运行 action 来启用：

![Re-run in debug mode](https://raw.githubusercontent.com/prefix-dev/setup-pixi/main/.github/assets/enable-debug-logging-light.png#only-light)
![Re-run in debug mode](https://raw.githubusercontent.com/prefix-dev/setup-pixi/main/.github/assets/enable-debug-logging-dark.png#only-dark)

!!!tip "调试日志文档"
    有关 GitHub Actions 中调试日志的更多信息，请参阅[官方文档](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging)。

#### Pixi 的调试日志

第二种是 pixi 可执行文件的调试日志。
可以通过设置 `log-level` 输入来指定。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    log-level: vvv # (1)!
```

1.  `q`、`default`、`v`、`vv` 或 `vvv` 之一

如果未指定，`log-level` 将默认为 `default` 或 `vv`，具体取决于[是否为 action 启用了调试日志](#debug-logging-of-the-action)。

### 自托管运行器

在自托管运行器上，可能会在 job 之间保留一些文件。
这可能导致问题或在 job 运行之间泄露密钥。
为此，你可以使用 `post-cleanup` 输入来指定 action 的清理行为（即在你执行完所有命令后*发生*什么）。

如果将 `post-cleanup` 设置为 `true`，action 将删除以下内容：

- `.pixi` 环境
- pixi 二进制文件
- rattler 缓存
- `~/.rattler` 中的其他 rattler 文件

如果未指定，`post-cleanup` 默认为 `true`。

在自托管运行器上，你可能还想将默认 pixi 安装位置更改为临时位置。你可以使用 `pixi-bin-path: ${{ runner.temp }}/bin/pixi` 来执行此操作。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    post-cleanup: true
    pixi-bin-path: ${{ runner.temp }}/bin/pixi # (1)!
```

1.  在 Windows 上为 `${{ runner.temp }}\Scripts\pixi.exe`

你也可以使用运行器上预安装的本地版本的 pixi，方法是不要设置任何 `pixi-version`、`pixi-url` 或 `pixi-bin-path` 输入。然后此 action 将在运行器的 PATH 中查找本地版本的 pixi。

### 使用 `pyproject.toml` 作为 pixi 的 manifest 文件

如果 `pyproject.toml` 包含 `[tool.pixi.workspace]` 部分且没有 `pixi.toml`，`setup-pixi` 将自动获取 `pyproject.toml`。
这可以通过设置 `manifest-path` 输入参数来覆盖。

```yaml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    manifest-path: pyproject.toml
```

### Monorepo 的工作目录

如果你正在处理 monorepo 且你的 pixi 项目在子目录中，你可以使用 `working-directory` 输入来指定 pixi 应该在何处查找 manifest 文件（`pixi.toml` 或 `pyproject.toml`）。

```yml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    working-directory: ./packages/my-project
```

这将使 pixi 在 `./packages/my-project` 目录而不是仓库根目录中查找 `pixi.toml` 或 `pyproject.toml`。所有 pixi 命令都将从此工作目录执行。

!!!note "也在其他步骤中"
    `working-directory` 输入只会影响由 `setup-pixi` 本身运行的命令。
    对于后续的 `run:` 步骤，你需要单独设置工作目录：

    ```yml
    - run: pixi run test
      working-directory: ./packages/my-project
    ```

如果需要，你可以将 `working-directory` 与 `manifest-path` 组合使用：

```yml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    working-directory: ./packages/my-project
    manifest-path: custom-pixi.toml
```

### 仅安装 pixi

如果你只想安装 pixi 而不想安装当前工作区，你可以使用 `run-install` 选项。

```yml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    run-install: false
```

### 从自定义 URL 下载 pixi

你也可以通过设置 `pixi-url` 输入参数从自定义 URL 下载 pixi。
你还可以选择将其与 `pixi-url-headers` 输入参数结合使用，为下载请求提供附加头，例如 bearer token。

```yml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    pixi-url: https://pixi-mirror.example.com/releases/download/v0.48.0/pixi-x86_64-unknown-linux-musl
    pixi-url-headers: '{"Authorization": "Bearer ${{ secrets.PIXI_MIRROR_BEARER_TOKEN }}"}'
```

`pixi-url` 输入参数也可以是 [Handlebars](https://handlebarsjs.com/) 模板字符串。
它将使用以下变量进行渲染：

- `version`：正在安装的 pixi 版本（`latest` 或类似 `v0.48.0` 的版本）。
- `latest`：一个布尔值，指示版本是否为 `latest`。
- `pixiFile`：要下载的 pixi 二进制文件的名称，由运行器的系统确定（例如 `pixi-x86_64-unknown-linux-musl`）。

默认情况下，`pixi-url` 等同于以下模板：

```yml
- uses: prefix-dev/setup-pixi@v0.9.4
  with:
    pixi-url: |
      {{#if latest~}}
      https://github.com/prefix-dev/pixi/releases/latest/download/{{pixiFile}}
      {{~else~}}
      https://github.com/prefix-dev/pixi/releases/download/{{version}}/{{pixiFile}}
      {{~/if}}
```

### 从环境变量设置输入

除了在 `with` 部分设置输入外，你还可以使用环境变量设置每个输入。
相应的环境变量名称是通过将输入名称转换为大写、将连字符替换为下划线，并在前面加上 `SETUP_PIXI_` 派生的。

例如，`pixi-bin-path` 输入可以使用 `SETUP_PIXI_PIXI_BIN_PATH` 环境变量设置。

如果执行 action 的自托管运行器，这特别有用。

输入始终优先于环境变量。

## 更多示例

如果你想查看更多示例，你可以查看 [`setup-pixi` 仓库的 GitHub Workflows](https://github.com/prefix-dev/setup-pixi/blob/main/.github/workflows/test.yml)。
