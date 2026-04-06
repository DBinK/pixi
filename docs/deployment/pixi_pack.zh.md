[`pixi-pack`](https://github.com/quantco/pixi-pack) 是一个简单的工具，它接受一个环境并将其打包成可以发送到目标机器的压缩归档文件。相应的 `pixi-unpack` 工具可用于解压缩归档文件并重建环境。

两个工具都可以通过以下方式安装：

```bash
pixi global install pixi-pack pixi-unpack
```

或者从 [releases 页面](https://github.com/Quantco/pixi-pack/releases) 下载预构建的二进制文件。

除了全局安装 `pixi-pack` 和 `pixi-unpack`，你也可以使用 `pixi exec` 在临时环境中运行它们：

```bash
pixi exec pixi-pack
pixi exec pixi-unpack environment.tar
```

!!!note ""
    如果你已经安装了 `pixi`、`pixi-pack` 和 `pixi-unpack`，也可以写 `pixi pack`（和 `pixi unpack`）。

![pixi-pack demo](https://raw.githubusercontent.com/quantco/pixi-pack/refs/heads/main/.github/assets/demo/demo-light.gif#only-light)
![pixi-pack demo](https://raw.githubusercontent.com/quantco/pixi-pack/refs/heads/main/.github/assets/demo/demo-dark.gif#only-dark)

你可以使用以下命令打包环境：

```bash
pixi-pack --environment prod --platform linux-64 pixi.toml
```

这将创建一个包含创建环境所需所有 conda 包的 `environment.tar` 文件。

```plain
# environment.tar
| pixi-pack.json
| environment.yml
| channel
|    ├── noarch
|    |    ├── tzdata-2024a-h0c530f3_0.conda
|    |    ├── ...
|    |    └── repodata.json
|    └── linux-64
|         ├── ca-certificates-2024.2.2-hbcca054_0.conda
|         ├── ...
|         └── repodata.json
```

### `pixi-unpack`：解包环境

使用 `pixi-unpack environment.tar`，你可以在目标系统上解包环境。
这将在 `./env` 目录中创建一个新的 conda 环境，其中包含你的 `pixi.toml` 中指定的所有包。
它还会创建 `activate.sh`（Windows 上为 `activate.bat`）文件，让你可以在没有安装 `conda` 或 `micromamba` 的情况下激活环境。

```bash
$ pixi-unpack environment.tar
$ ls
env/
activate.sh
environment.tar
$ cat activate.sh
export PATH="/home/user/project/env/bin:..."
export CONDA_PREFIX="/home/user/project/env"
. "/home/user/project/env/etc/conda/activate.d/activate_custom_package.sh"
```

### 跨平台包

由于 `pixi-pack` 只是从 conda 仓库下载 `.conda` 和 `.tar.bz2` 文件，你可以轻松地为不同平台创建包。

```bash
pixi-pack --platform win-64
```

!!! note
    你只能在与创建包时相同平台的系统上解包。

### 自解压二进制文件

你可以创建一个包含打包环境和解包脚本的自解压二进制文件。
当你想要将环境分发给没有安装 `pixi-unpack` 的用户时，这会很有用。

=== "Linux & macOS"
    ```bash
    $ pixi-pack --create-executable
    $ ls
    environment.sh
    $ ./environment.sh
    $ ls
    env/
    activate.sh
    environment.sh
    ```

=== "Windows"
    ```powershell
    PS > pixi-pack --create-executable
    PS > ls
    environment.ps1
    PS > .\environment.ps1
    PS > ls
    env/
    activate.sh
    environment.ps1
    ```

#### 自定义 pixi-unpack 可执行文件路径

创建自解压二进制文件时，你可以指定自定义路径或 URL 到 `pixi-unpack` 可执行文件，以避免从[默认位置](https://github.com/Quantco/pixi-pack/releases/latest)下载它。

你可以提供以下任一作为 `--pixi-unpack-source`：

- `pixi-unpack` 可执行文件的 URL，如 `https://my.mirror/pixi-pack/pixi-unpack-x86_64-unknown-linux-musl`
- `pixi-unpack` 二进制文件的路径，如 `./pixi-unpack-x86_64-unknown-linux-musl`

##### 使用示例

使用 URL：

```bash
pixi-pack --create-executable --pixi-unpack-source https://my.mirror/pixi-pack/pixi-unpack-x86_64-unknown-linux-musl
```

使用路径：

```bash
pixi-pack --create-executable --pixi-unpack-source ./pixi-unpack-x86_64-unknown-linux-musl
```

!!! note

    生成的的可执行文件是一个简单的脚本文件，其中包含 `pixi-unpack` 二进制文件和打包的环境。

### 注入额外的包

你可以使用 `--inject` 标志将未在 `pixi.lock` 中指定的额外包注入到环境中：

```bash
pixi-pack --inject local-package-1.0.0-hbefa133_0.conda pixi.toml
```

这在你构建包本身并希望将构建的包包含在环境中但仍想使用工作区的 `pixi.lock` 时特别有用。

### PyPI 支持

你也可以将 PyPI wheel 包打包到你的环境中。
`pixi-pack` 只支持 wheel 包，不支持源码分发。
如果你使用源码分发，可以通过使用 `--ignore-pypi-non-wheel` 标志忽略它们。
这将跳过作为源码分发的 PyPI 包的打包。

`--inject` 选项也支持 wheel。

```bash
pixi-pack --ignore-pypi-non-wheel --inject my_webserver-0.1.0-py3-none-any.whl
```

!!! warning

    与从 conda 包注入不同，我们无法验证注入的 wheel 是否与目标环境兼容。请确保包是兼容的。

### 镜像和 S3 中间件

你可以通过创建配置文件并使用 `--config` 引用来使用镜像中间件，详见 [pixi 文档](../reference/pixi_configuration.md#mirror-configuration)。

```toml title="config.toml"
[mirrors]
"https://conda.anaconda.org/conda-forge" = ["https://my.artifactory/conda-forge"]
```

如果你在 pixi 中使用 [S3](./s3.md)，也可以在配置文件中添加适当的 S3 配置并引用它。

```toml title="config.toml"
[s3-options.my-s3-bucket]
endpoint-url = "https://s3.eu-central-1.amazonaws.com"
region = "eu-central-1"
force-path-style = false
```

### 设置最大并行下载数

```toml
[concurrency]
downloads = 5
```

使用 `pixi-pack --config config.toml` 使用自定义配置文件。
详见 [pixi 文档](../reference/pixi_configuration.md#concurrency)。

### 缓存下载的包

你可以使用 `--use-cache` 标志缓存下载的包以加快后续打包操作：

```bash
pixi-pack --use-cache ~/.pixi-pack/cache
```

这会将所有下载的包存储在指定目录中，并在未来的打包操作中重用它们。缓存遵循与 conda 通道相同的结构，按平台子目录组织包（例如 linux-64、win-64 等）。

在以下情况下使用缓存特别有用：

- 创建具有重叠依赖的多个包
- 处理需要下载时间的大包
- 在带宽有限的机器上操作
- 运行 CI/CD 流水线，包缓存可以显著缩短构建时间

### 无需 pixi-pack 解包

如果你的目标系统上没有 `pixi-pack`，也不想使用自解压二进制文件（见上文），只要有 `conda` 或 `micromamba` 仍然可以安装环境。
只需解压缩 `environment.tar`，然后你的系统上就有一个本地通道，所有必需的包都可用。
在这个本地通道旁边，你会找到一个 `environment.yml` 文件，其中包含环境规格。
然后你可以使用 `conda` 或 `micromamba` 安装环境：

```bash
tar -xvf environment.tar
micromamba create -p ./env --file environment.yml
# 或
conda env create -p ./env --file environment.yml
```

!!! note

    `environment.yml` 和 `repodata.json` 文件仅用于此用例，`pixi-unpack` 不使用它们。

!!! note

    `conda` 和 `mamba` 在安装 python 时总是会同时安装 pip，参见 [`conda` 文档](https://docs.conda.io/projects/conda/en/25.1.x/user-guide/configuration/settings.html#add-pip-as-python-dependency-add-pip-as-python-dependency)。
    这与 `pixi` 的工作方式不同，并可能导致使用 `pixi-pack` 兼容性模式时的求解器错误，因为 `pixi` 默认不包含 `pip`。
    你可以通过两种方式解决此问题：

    - 使用 `pixi add pip` 将 `pip` 添加到你的 `pixi.lock` 文件中。
    - 配置 `conda`（或 `mamba`）默认不安装 `pip`，运行 `conda config --set add_pip_as_python_dependency false`（或在你的 `~/.condarc` 中添加 `add_pip_as_python_dependency: False`）
