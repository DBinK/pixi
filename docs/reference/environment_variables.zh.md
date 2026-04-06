
## 可配置的环境变量

Pixi 也可以通过环境变量配置。

<table>
  <thead>
    <tr>
      <th>名称</th>
      <th>描述</th>
      <th>默认值</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>PIXI_HOME</code></td>
      <td>定义 pixi 放置其全局数据的目录。</td>
      <td><a href="https://docs.rs/dirs/latest/dirs/fn.home_dir.html">HOME</a>/.pixi</td>
    </tr>
    <tr>
      <td><code>PIXI_CACHE_DIR</code></td>
      <td>定义 pixi 放置其缓存的目录。</td>
      <td>
        <ul>
          <li>如果未设置 <code>PIXI_CACHE_DIR</code>，则使用 <code>RATTLER_CACHE_DIR</code> 环境变量。</li>
          <li>如果未设置，则当目录存在时使用 <code>XDG_CACHE_HOME/pixi</code>。</li>
          <li>如果未设置，则使用 <a href="https://docs.rs/rattler/latest/rattler/fn.default_cache_dir.html">rattler::default_cache_dir</a> 的默认缓存目录。</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td><code>RATTLER_AUTH_FILE</code></td>
      <td>覆盖凭据文件的默认位置。设置后，这是 pixi 使用的唯一认证数据来源。请参阅<a href="../../deployment/authentication/#override-the-authentication-storage">认证文档</a>了解文件格式。</td>
      <td>
        <ul>
          <li>如果未设置 <code>RATTLER_AUTH_FILE</code>，则使用系统密钥链。</li>
          <li>如果没有可用的密钥链，则使用来自 <code>~/.rattler/credentials.json</code> 的文件存储。</li>
          <li>此外，还会检查 <code>.netrc</code> 文件认证。</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

## Pixi 设置的环境变量

当使用 `pixi run`、`pixi shell` 或 `pixi shell-hook` 命令时，Pixi 会设置以下环境变量：

- `PIXI_PROJECT_ROOT`：项目的根目录。
- `PIXI_PROJECT_NAME`：项目的名称。
- `PIXI_PROJECT_MANIFEST`：清单文件（`pixi.toml`）的路径。
- `PIXI_PROJECT_VERSION`：项目的版本。
- `PIXI_PROMPT`：shell 中使用的提示符，也由 `pixi shell` 本身使用。
- `PIXI_ENVIRONMENT_NAME`：环境的名称，默认为 `default`。
- `PIXI_ENVIRONMENT_PLATFORMS`：项目支持的平台的逗号分隔列表。
- `CONDA_PREFIX`：环境的路径。（用于已经理解 conda 环境的多个工具）
- `CONDA_DEFAULT_ENV`：环境的名称。（用于已经理解 conda 环境的多个工具）
- `PATH`：我们将环境的 `bin` 目录预先添加到 `PATH` 变量，这样你可以直接使用环境中安装的工具。
- `INIT_CWD`：仅在 `pixi run` 中：运行命令的目录。

!!! note
    即使这些是环境变量，它们也无法被覆盖。例如，你不能通过在环境中设置 `PIXI_PROJECT_ROOT` 来更改项目的根目录。

## 环境变量优先级

以下优先级规则适用于环境变量：`task.env` > `activation.env` > `activation.scripts` > 依赖项的激活脚本 > 外部环境变量。
在较高优先级定义的变量将覆盖在较低优先级定义的变量。

!!! warning

    在旧版本的 Pixi 中，这个优先级没有得到很好的定义，并且存在一些已知的
    与当前优先级存在的偏差。请参阅[高级任务文档](../workspace/advanced_tasks.md#environment-variables)
    中的警告以获取更多详细信息和迁移指南。

##### 示例 1：`task.env` > `activation.env`

在 `pixi.toml` 中，我们在 `tasks.hello` 和 `activation.env` 中都定义了环境变量 `HELLO_WORLD`。

当我们运行 `echo $HELLO_WORLD` 时，它将输出：
```
Hello world!
```

```toml title="pixi.toml"
[tasks.hello]
cmd = "echo $HELLO_WORLD"
env = { HELLO_WORLD = "Hello world!" }
[activation.env]
HELLO_WORLD = "Activate!"
```

##### 示例 2：`activation.env` > `activation.scripts`

在 `pixi.toml` 中，我们在 `activation.env` 和激活脚本文件 `setup.sh` 中都定义了相同的环境变量 `DEBUG_MODE`。
当我们运行 `echo Debug mode: $DEBUG_MODE` 时，它将输出：
```bash
Debug mode: enabled
```

```toml title="pixi.toml"
[activation.env]
DEBUG_MODE = "enabled"

[activation]
scripts = ["setup.sh"]
```

```bash title="setup.sh"
export DEBUG_MODE="disabled"
```

##### 示例 3：`activation.scripts` > 依赖项的激活脚本

在 `pixi.toml` 中，我们有本地激活脚本和一个依赖项 `my-package`，它也通过其激活脚本设置环境变量。
当我们运行 `echo Library path: $LIB_PATH` 时，它将输出：
```
Library path: /my/lib
```

```toml title="pixi.toml"
[activation]
scripts = ["local_setup.sh"]

[dependencies]
my-package = "*"  # 此包有自己的激活脚本，设置 LIB_PATH="/dep/lib"
```
```bash title="local_setup.sh"
export LIB_PATH="/my/lib"
```

##### 示例 4：依赖项的激活脚本 > 外部环境变量

如果我们有一个设置 `PYTHON_PATH` 的依赖项，并且相同的变量已在外部环境中设置。
当我们运行 `echo Python path: $PYTHON_PATH` 时，它将输出：
```bash
Python path: /pixi/python
```
```
# 外部环境
export PYTHON_PATH="/system/python"
```
```toml title="pixi.toml"
[dependencies]
python-utils = "*"  # 此包在其激活脚本中设置 PYTHON_PATH="/pixi/python"
```

##### 示例 5：复杂示例 - 所有优先级组合
在 `pixi.toml` 中，我们在多个级别定义相同的变量 `APP_CONFIG`：
```toml title="pixi.toml"
[tasks.start]
cmd = "echo Config: $APP_CONFIG"
env = { APP_CONFIG = "task-specific" }

[activation.env]
APP_CONFIG = "activation-env"

[activation]
scripts = ["app_setup.sh"]

[dependencies]
config-loader = "*"  # 设置 APP_CONFIG="dependency-config"
```
```bash title="app_setup.sh"
export APP_CONFIG="activation-script"
```
```bash
# 外部环境
export APP_CONFIG="system-config"
```

由于 `task.env` 具有最高优先级，当我们运行 `pixi run start` 时，它将输出：

```
Config: task-specific
```
