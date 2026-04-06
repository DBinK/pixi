!!!tip "`conda-deny` 一条命令："
    在你喜欢的 `pixi` 工作区中运行：
    ```bash
    pixi exec conda-deny check --osi
    ```

    这将根据 [OSI 批准的许可证](https://opensource.org/licenses) 列表检查你的工作区的许可证合规性。

[conda-deny](https://github.com/Quantco/conda-deny) 是一个用于检查软件环境依赖项许可证合规性的 CLI 工具。合规性是根据用户提供的许可证白名单进行检查的。

### 💿 安装
你可以使用 `pixi` 安装 `conda-deny`：

```bash
pixi global install conda-deny
```

或者从 [releases 页面](https://github.com/quantco/conda-deny/releases) 下载预构建的二进制文件。

### 🎯 使用方法

![conda-deny demo](https://raw.githubusercontent.com/Quantco/conda-deny/refs/heads/main/.github/assets/demo/demo-light.gif#gh-light-mode-only)
![conda-deny demo](https://raw.githubusercontent.com/Quantco/conda-deny/refs/heads/main/.github/assets/demo/demo-dark.gif#gh-dark-mode-only)

`conda-deny` 可以在你的 `pixi.toml` 或 `pyproject.toml` 中配置（首选 `pixi.toml`）。
该工具期望以下格式的配置：

```toml
[tool.conda-deny]
#--------------------------------------------------------
# 常规设置选项：
#--------------------------------------------------------
license-allowlist = "https://raw.githubusercontent.com/quantco/conda-deny/main/tests/test_remote_base_configs/conda-deny-license_allowlist.toml" # 或 ["license_allowlist.toml", "other_license_allowlist.toml"]
platform = "linux-64" # 或 ["linux-64", "osx-arm64"]
environment = "default" # 或 ["default", "py39", "py310", "prod"]
lockfile = "environment/pixi.lock" # 或 ["environment1/pixi.lock", "environment2/pixi.lock"]
# lockfile 也支持 glob 模式：
# lockfile = "environments/**/*.lock"

#--------------------------------------------------------
# 直接在配置文件中设置许可证白名单：
#--------------------------------------------------------
safe-licenses = ["MIT", "BSD-3-Clause"]
ignore-packages = [
    { package = "make", version = "0.1.0" },
]
```

安装后，你可以在工作区中运行 `conda-deny check`。
这将检查你的 `pixi.lock` 中定义的依赖项是否符合你的白名单。

### 🔒 受限访问白名单

如果需要 Bearer Token 来访问你的白名单，你可以使用 `CONDA_DENY_BEARER_TOKEN` 提供它。
一个示例用例是包含你的白名单的私有仓库。

### 输出格式

`conda-deny` 支持通过 `--output`（或 `-o` 标志）使用不同的输出格式。
输出格式化同时适用于 `list` 和 `check` 命令。

=== "CSV"
    ```bash
    $ conda-deny list --output csv
    package_name,version,license,platform,build,safe
    _openmp_mutex,4.5,BSD-3-Clause,linux-aarch64,2_gnu,false
    _openmp_mutex,4.5,BSD-3-Clause,linux-64,2_gnu,false
    ...
    ```


=== "JSON"
    ```bash
    $ conda-deny list --output json-pretty
    {
    "unsafe": [
        {
        "build": "conda_forge",
        "license": {
            "Invalid": "None"
        },
        "package_name": "_libgcc_mutex",
        "platform": "linux-64",
        "version": "0.1"
        },
        {
        "build": "h57d6b7b_14",
        "license": {
            "Invalid": "LGPL-2.0-or-later AND LGPL-2.0-or-later WITH exceptions AND GPL-2.0-or-later AND MPL-2.0"
        },
        "package_name": "_sysroot_linux-aarch64_curr_repodata_hack",
        "platform": "noarch",
        "version": "4"
        },
    ...
    ```

!!!tip 创建许可证捆绑包
    通过运行 `conda-deny bundle`，`conda-deny` 将创建一个包含你所有依赖项原始许可证文件的目录。

    这在创建 SBOM 或与他人共享合规信息时会很方便。
