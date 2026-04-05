# 高级任务

构建包时，你通常需要做的不仅仅是运行代码。
格式化、linting、编译、测试、基准测试等步骤通常是 workspace 的一部分。
使用 Pixi 任务，这应该变得更容易。

以下是一些快速示例

```toml title="pixi.toml"
[tasks]
# Commands as lists so you can also add documentation in between.
configure = { cmd = [
    "cmake",
    # Use the cross-platform Ninja generator
    "-G",
    "Ninja",
    # The source is in the root directory
    "-S",
    ".",
    # We wanna build in the .build directory
    "-B",
    ".build",
] }

# Add task descriptions, to be surfaced when the tasks are listed
say-hello = { cmd = ["echo", "hello world"], description = "Greet the world." }

# Depend on other tasks
build = { cmd = ["ninja", "-C", ".build"], depends-on = ["configure"] }

# Using environment variables
run = "python main.py $PIXI_PROJECT_ROOT"
set = "export VAR=hello && echo $VAR"

# Cross platform file operations
copy = "cp pixi.toml pixi_backup.toml"
clean = "rm pixi_backup.toml"
move = "mv pixi.toml backup.toml"

# Setting a default environment for the task
test = { cmd = "pytest", default-environment = "test" }
```

## 依赖

就像包可以依赖其他包一样，我们的任务可以依赖其他任务。
这允许用一个命令运行完整的管道。

一个明显的例子是**编译**后再**运行**应用程序。

查看我们的 [`cpp_sdl` 示例](https://github.com/prefix-dev/pixi/tree/main/examples/cpp-sdl)以获取运行示例。
在该包中，我们有一些相互依赖的任务，这样当你运行 `pixi run start` 时，一切都会按预期设置。

```fish
pixi task add configure "cmake -G Ninja -S . -B .build"
pixi task add build "ninja -C .build" --depends-on configure
pixi task add start ".build/bin/sdl_example" --depends-on build
```

结果是将以下行添加到 `pixi.toml`

```toml title="pixi.toml"
[tasks]
# Configures CMake
configure = "cmake -G Ninja -S . -B .build"
# Build the executable but make sure CMake is configured first.
build = { cmd = "ninja -C .build", depends-on = ["configure"] }
# Start the built executable
start = { cmd = ".build/bin/sdl_example", depends-on = ["build"] }
```

任务将按顺序执行：

- 首先执行 `configure`，因为它没有依赖。
- 然后执行 `build`，因为它只依赖于 `configure`。
- 然后执行 `start`，因为它的所有依赖都已运行。

如果其中一个命令失败（退出非零代码），它将停止，下一个不会启动。

利用这个逻辑，你也可以创建别名，因为你不必在任务中指定任何命令。

```shell
pixi task add fmt ruff
pixi task add lint pylint
```

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/pixi_task_alias.toml:not-all"
```

!!! tip "隐藏任务"
    任务可以通过[命名](#task-names)时使用 `_` 前缀来隐藏，不显示在面向用户的命令中。

### 简写语法

Pixi 支持一种简写语法来定义仅依赖于其他任务的任务。不使用更详细的 `depends-on` 字段，你可以直接将任务定义为依赖数组。

执行：

```
pixi task alias style fmt lint
```

结果如下 `pixi.toml`:

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/pixi_task_alias.toml:all"
```

现在你可以用一个命令运行两个工具。

```shell
pixi run style
```

### 任务依赖的环境规范

你可以指定依赖任务使用的环境：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/tasks_depends_on.toml:tasks"
```

这允许你在单个管道中的不同环境中运行任务。
当你运行主任务时，Pixi 确保每个依赖任务使用其指定的环境：

```shell
pixi run test-all
```

为任务依赖指定的环境优先于通过 CLI `--environment` 标志指定的环境。
这意味着即使你运行 `pixi run test-all --environment py312`，第一个依赖仍然会在 `py311` 环境中运行，如 TOML 文件中所指定。

在上面的例子中，`test-all` 任务在 Python 3.11 和 3.12 环境中运行 `test` 任务，允许你通过单个命令验证不同 Python 版本的兼容性。

## 工作目录

Pixi 任务支持定义工作目录。

`cwd` 代表 Current Working Directory。
该目录相对于 Pixi workspace 根目录，即 `pixi.toml` 文件所在的位置。

默认情况下，任务从 Pixi workspace 根目录执行。
要更改此内容，请使用 `--cwd` 标志。
例如，考虑以下结构的 Pixi workspace：

```shell
├── pixi.toml
└── scripts
    └── bar.py
```

要添加一个运行 `scripts` 目录中 `bar.py` 文件的任务，请使用：

```shell
pixi task add bar "python bar.py" --cwd scripts
```

这将以下内容添加到 [manifest 文件](../reference/pixi_manifest.md)：

```toml title="pixi.toml"
[tasks]
bar = { cmd = "python bar.py", cwd = "scripts" }
```

## 默认环境

你可以使用 `default-environment` 字段设置任务使用的默认 Pixi [environment](../tutorials/multi_environment.md#adding-an-environment)：
```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_default_environment.toml:default-environment"
```

默认环境可以像往常一样使用 `--environment` 参数覆盖：
```shell
pixi run -e other_environment test
```

## 任务参数

任务可以接受可以在命令中引用的参数。这为你的任务提供了更多的灵活性和可重用性。

### 为什么使用任务参数？

任务参数使你的任务更加通用和可维护：

- **可重用性**：创建可以处理不同输入的通用任务，而不是为每个特定情况复制任务
- **灵活性**：在运行时更改行为，而无需修改你的 pixi.toml 文件
- **清晰性**：通过明确定义哪些值可以自定义来使你的任务意图明确
- **验证**：定义必需参数以确保任务被正确调用
- **默认值**：设置合理的默认值，同时允许在需要时覆盖

例如，不要为开发和生产模式创建单独的任务，你可以创建一个处理两种情况的参数化任务。

参数可以是：

- **必需**：运行任务时必须提供
- **可选**：可以有默认值，未明确提供时使用
- **受限**：可以使用 `choices` 限制为允许的值集

### 定义任务参数

在任务中使用 `args` 字段定义参数：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_arguments.toml:project_tasks"
```

!!! note "参数命名限制"
    参数名称不能包含破折号（` - `），因为它们在 MiniJinja 中被视为减号符号。改用下划线（`_`）或 camelCase。

### 使用任务参数

运行任务时，按定义顺序提供参数：

```shell
# Required argument
pixi run greet John
✨ Pixi task (greet in default): echo Hello, John!

# Default values are used when omitted
pixi run build
✨ Pixi task (build in default): echo Building my-app in development mode

# Override default values
pixi run build my-project production
✨ Pixi task (build in default): echo Building my-project in production mode

# Mixed argument types
pixi run deploy auth-service
✨ Pixi task (deploy in default): echo Deploying auth-service to staging
pixi run deploy auth-service production
✨ Pixi task (deploy in default): echo Deploying auth-service to production
```
### 使用 `--` 传递额外参数

当任务定义类型化 `args` 时，所有命令行值都与这些定义匹配。如果需要在类型化参数之上将*额外*标志或参数直接传递给底层命令，请使用 `--` 作为分隔符。`--` 之后的任何内容都会按原样转发给命令，无论任务的 `args` 定义如何。

```toml
[tasks.test]
cmd = "pytest {{ target }} -v"
args = [{ arg = "target", default = "tests/unit" }]
```

```shell
# Without --, extra flags would cause an error:
# × task 'test' received more arguments than expected
#   hint: use `--` to separate task arguments from extra passthrough arguments
pixi run test tests/integration --tb=short --maxfail=5

# With --, the typed arg is filled and the rest is forwarded:
pixi run test tests/integration -- --tb=short --maxfail=5
✨ Pixi task (test in default): pytest tests/integration -v --tb=short --maxfail=5
```

你也可以在依赖默认参数值时使用 `--`：

```shell
# "target" uses its default, "--maxfail=5" is forwarded to the command
pixi run test -- --maxfail=5
✨ Pixi task (test in default): pytest tests/unit -v --maxfail=5
```

!!! note "没有类型化参数的任务"
    对于**不**定义 `args` 的任务，`--` 会按原样传递给底层命令。这为使用 `--` 本身的程序（如 `git log -- somefile`）保留其含义。

### 使用 Choices 限制值

你可以使用 `choices` 字段限制参数的允许值。如果提供了不在列表中的值，Pixi 将报告错误而不是运行任务。

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_arguments.toml:project_tasks_choices"
```

```shell
# Providing a valid choice
pixi run compile debug
✨ Pixi task (compile): echo 'Compiling in debug mode'
Compiling in debug mode

# Default value must also be one of the choices
pixi run test
✨ Pixi task (test): echo 'Running unit tests'
Running unit tests

# Providing an invalid value results in an error
pixi run compile fast
× got 'fast' for argument 'mode' of task 'compile', choose from: debug, release
```

### 将参数传递给依赖任务

你可以将参数传递给作为其他任务依赖的任务：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_arguments_dependent.toml:project_tasks"
```

执行依赖任务时，参数会传递给依赖：

```shell
pixi run install-release
✨ Pixi task (install in default): echo Installing with manifest /path/to/manifest and flag --debug

pixi run deploy
✨ Pixi task (install in default): echo Installing with custom/path and flag --verbose
✨ Pixi task (deploy in default): echo Deploying
```

当依赖任务未指定所有参数时，缺失的参数使用默认值：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_arguments_partial.toml:project_tasks"
```

```shell
pixi run partial-override
✨ Pixi task (base-task in default): echo Base task with override1 and default2
```

要允许依赖任务接受传递给依赖的参数，你可以使用与向命令传递参数相同的语法：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_arguments_partial.toml:project_tasks_with_arg"
```

```shell
pixi run partial-override-with-arg
✨ Pixi task (base-task in default): echo Base task with override1 and new-default2
pixi run partial-override-with-arg cli-arg
✨ Pixi task (base-task in default): echo Base task with override1 and cli-arg
```

### 任务参数的 MiniJinja 模板

manifest 中定义的任务命令和环境变量支持 MiniJinja 模板语法来访问和格式化参数值。这在构造命令时提供了强大的灵活性。

!!! note "模板和临时 CLI 命令"
    MiniJinja 模板仅应用于 manifest 文件（`pixi.toml` / `pyproject.toml`）中定义的任务。
    直接传递给命令行 `pixi run` 的临时命令默认**不**进行模板化，
    因此像 `pixi run echo '{{ hello }}'` 这样的命令会按原样传递。
    使用 `--templated` 标志选择加入 CLI 命令的模板渲染：

    ```shell
    pixi run --templated echo '{{ pixi.platform }}'
    ```

在命令中使用参数的基本语法：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_minijinja_simple.toml:tasks"
```

你也可以使用过滤器来转换参数值：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_workspaces/minijinja/task_args/pixi.toml:tasks"
```

#### Pixi 变量

除了任务参数，Pixi 还在 MiniJinja 上下文中自动提供一个包含系统变量的 `pixi` 对象：

| 变量 | 说明 | 示例值 |
|----------|-------------|---------------|
| `pixi.platform` | 任务将运行的环境的平台名称 | `linux-64`, `osx-arm64`, `win-64` |
| `pixi.environment.name` | 当前环境的名称（当可用时） | `default`, `prod`, `test` |
| `pixi.manifest_path` | manifest 文件的绝对路径 | `/path/to/project/pixi.toml` |
| `pixi.version` | 使用的 pixi 版本 | `0.59.0` |
| `pixi.init_cwd` | 调用 pixi 时的当前工作目录 | `/path/to/cwd` |
| `pixi.is_win` | 指示平台是否为 Windows 的布尔标志 | `true` 或 `false` |
| `pixi.is_unix` | 指示平台是否为类 Unix 的布尔标志 | `true` 或 `false` |
| `pixi.is_linux` | 指示平台是否为 Linux 的布尔标志 | `true` 或 `false` |
| `pixi.is_osx` | 指示平台是否为 macOS 的布尔标志 | `true` 或 `false` |

这些变量对于创建平台特定或环境感知的任务特别有用：

```toml title="pixi.toml"
[tasks]
# Platform-specific commands
build = { cmd = "cargo build --target {{ pixi.platform }}", args = [] }
download-binary = { cmd = "curl -O https://example.com/binary-{{ pixi.platform }}.tar.gz", args = [] }

# Conditional execution based on platform
install = { cmd = "{% if pixi.is_win %}install.bat{% else %}./install.sh{% endif %}", args = [] }

# Environment-aware tasks
deploy = { cmd = "deploy.sh --env {{ pixi.environment.name }}", args = [] }

# Using manifest path
validate = { cmd = "validator --manifest {{ pixi.manifest_path }}", args = [] }

# Using init_cwd in inputs/outputs for caching
mkdir_test = { cmd = "mkdir -p {{ pixi.init_cwd }}/test", outputs = ["{{ pixi.init_cwd }}/test"] }
```

pixi 变量也可以与任务参数结合使用：

```toml title="pixi.toml"
[tasks]
deploy = {
    cmd = "deploy.sh --platform {{ pixi.platform }} --env {{ environment }} --version {{ pixi.version }}",
    args = [{ arg = "environment", default = "staging" }]
}
```

当运行具有类型化参数的任务时，平台会自动反映任务执行环境的最佳平台。

有关可用过滤器和模板语法的更多信息，请参阅 [MiniJinja 文档](https://docs.rs/minijinja/latest/minijinja/filters/index.html)。

## 任务名称

任务名称遵循以下规则：

- 名称中**不允许有空格**。
- 在表中必须**唯一**。
- 以 [`_`]("下划线") 开头的任务名称将**隐藏**任务，使其不出现在 `pixi task list` 命令中。

如果你的 workspace 定义了许多任务但用户只需要使用其中一部分，隐藏任务会很有用。

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/task_visibility.toml:project_tasks"
```

## 缓存

当你为任务指定 `inputs` 和/或 `outputs` 时，Pixi 将重用任务的结果。

对于缓存，Pixi 检查以下条件是否为真：

- 环境中没有包发生变化。
- 选择的输入和输出与上次运行任务时相同。我们计算所有由 globs 选择的文件的指纹，并将它们与上次任务运行时进行比较。
- 命令与上次任务运行时相同。

如果所有这些条件都满足，Pixi 不会再次运行任务，而是使用现有结果。

输入和输出可以指定为 globs，它们将展开为所有匹配的文件。你也可以在 `inputs` 和 `outputs` 字段中使用 MiniJinja 模板来参数化路径，使任务更可重用：

```toml title="pixi.toml"
--8<-- "docs/source_files/pixi_tomls/tasks_minijinja_inputs_outputs.toml:tasks"
```

在输入/输出中使用模板变量时，Pixi 使用提供的参数或环境变量展开模板，并使用解析的路径进行缓存决策。这允许你创建可以处理不同文件而无需复制任务配置的通用任务：

```shell
# First run processes the file and caches the result
pixi run process-file data1

# Second run with the same argument uses the cached result
pixi run process-file data1  # [cache hit]

# Run with a different argument processes a different file
pixi run process-file data2
```

注意：如果你想调试 globs，可以使用 `--verbose` 标志查看选择了哪些文件。

```shell
# shows info logs of all files that were selected by the globs
pixi run -v start
```

## 环境变量

你可以直接为任务设置环境变量，也可以通过其他方式设置。
有关设置环境变量的方法的完整细节以及它们如何相互影响，请参阅[环境变量优先级文档](../reference/environment_variables.md#environment-variable-priority)。

有关任务中环境变量的注意事项：
- 通过 `tasks.<name>.env` 设置的值在任务运行时由 `deno_task_shell` 解释。因此，shell 风格的展开（如 `env = { VAR = "$FOO" }`）在所有操作系统上都能正常工作。
- 模板允许在环境变量中使用，例如当你有 `{ cmd="pytest", env={ BACKEND="{{ backend }}" }, args=[{arg="backend", default="numpy"}] }` 时。`{{ backend }}` 参数值由传入的 `backend` 值解释。

!!! warning

    在旧版 Pixi 中，这个优先级没有很好地定义，存在一些已知偏差：

    - `activation.scripts` 曾经优先于 `activation.env`
    - 依赖的激活脚本曾经优先于 `activation.env`
    - 外部环境变量曾经覆盖在 `task.env` 中设置的变量

    如果你之前依赖的某个优先级不再适用，你可能需要更改任务定义。

    对于覆盖 `task.env` 与外部环境变量的特定情况，现在可以使用[任务参数](#task-arguments)重新创建此行为。例如，如果你之前使用这样的设置：

    ```toml title="pixi.toml"
    [tasks]
    echo = { cmd = "echo $ARGUMENT", env = { ARGUMENT = "hello" } }
    ```

    ```shell
    ARGUMENT=world pixi run echo
    ✨ Pixi task (echo in default): echo $ARGUMENT
    world
    ```

    你现在可以像这样重新创建行为：

    ```toml title="pixi.toml"
    [tasks]
    echo = { cmd = "echo {{ ARGUMENT }}", args = [{"arg" = "ARGUMENT", "default" = "hello" }] }
    ```

    ```shell
    pixi run echo world
    ✨ Pixi task (echo): echo world
    world
    ```

## 清洁环境

你可以确保任务的環境是"Pixi only"。
在這裡，Pixi 只会包含在你的平台上运行命令所需的最小环境变量。
环境将包含 conda 环境设置的所有变量，如 `"CONDA_PREFIX"`。
但它会包含一些来自 shell 的默认值，如：
`"DISPLAY"`、`"LC_ALL"`、`"LC_TIME"`、`"LC_NUMERIC"`、`"LC_MEASUREMENT"`、`"SHELL"`、`"USER"`、`"USERNAME"`、`"LOGNAME"`、`"HOME"`、`"HOSTNAME"`、`"TMPDIR"`、`"XPC_SERVICE_NAME"`、`"XPC_FLAGS"`

```toml
[tasks]
clean_command = { cmd = "python run_in_isolated_env.py", clean-env = true }
```
此设置也可以从命令行使用 `pixi run --clean-env TASK_NAME` 设置。

!!! warning "`clean-env` 在 Windows 上不支持"
    在 Windows 上很难创建"清洁环境"，因为 `conda-forge` 不提供 Windows 编译器，Windows 需要很多基础变量。
    实现这个功能是不值得的，因为大量边缘情况会使其无法使用。

## 我们的任务运行器：deno_task_shell

为了支持不同的操作系统（Windows、OSX 和 Linux），Pixi 集成了一个可以在所有平台上运行的 shell。
这是 [`deno_task_shell`](https://deno.land/manual@v1.35.0/tools/task_runner#built-in-commands)。
任务 shell 是 bourne-shell 接口的有限实现。
任务命令行和 `tasks.<name>.env` 的值由此 shell 解析和展开。

### 内置命令

除了运行实际可执行文件（如 `./myprogram`、`cmake` 或 `python`）外，shell 还有一些内置命令。

- `cp`: 复制文件。
- `mv`: 移动文件。
- `rm`: 删除文件或目录。
  例如：`rm -rf [FILE]...` - 通常用于递归删除文件或目录。
- `mkdir`: 创建目录。
  例如：`mkdir -p DIRECTORY...` - 通常用于创建目录及其所有父目录，目录存在时不会出错。
- `pwd`: 打印当前/工作目录的名称。
- `sleep`: 延迟指定的时间。
  例如：`sleep 1` 睡一秒，`sleep 0.5` 睡半秒，或 `sleep 1m` 睡一分钟
- `echo`: 显示一行文本。
- `cat`: 连接文件并在 stdout 输出它们。当未提供参数时，它读取并输出 stdin。
- `exit`: 使 shell 退出。
- `unset`: 取消设置环境变量。
- `xargs`: 从 stdin 构建参数并执行命令。

### 语法

- **布尔列表：**使用 `&&` 或 `||` 分隔两个命令。
  - `&&`：如果 `&&` 前的命令成功则继续下一个命令。
  - `||`：如果 `||` 前的命令失败则继续下一个命令。
- **顺序列表：**使用 `;` 运行两个命令，不检查第一个命令是失败还是成功。
- **环境变量：**
  - 使用设置环境变量：`export ENV_VAR=value`
  - 使用环境变量： `$ENV_VAR`
  - 取消设置环境变量：`unset ENV_VAR`
- **Shell 变量：**Shell 变量类似于环境变量，但不会导出到生成的命令。
  - 设置它们：`VAR=value`
  - 使用它们：`VAR=value && echo $VAR`
- **管道：**将命令的 stdout 输出到下一个命令的 stdin
  - `|`：`echo Hello | python receiving_app.py`
  - `|&`：使用它也获取 stderr 作为输入。
- **命令替换：** `$()` 使用命令的输出作为另一个命令的输入。
  - `python main.py $(git rev-parse HEAD)`
- **否定退出码：** `! ` 在任何命令前将退出码从 1 否定到 0 或反之亦然。
- **重定向：** `>` 将 stdout 重定向到文件。
  - `echo hello > file.txt` 会将 `hello` 放入 `file.txt` 并覆盖现有文本。
  - `python main.py 2> file.txt` 会将 `stderr` 输出放入 `file.txt`。
  - `python main.py &> file.txt` 会将 `stderr` **和** `stdout` 放入 `file.txt`。
  - `echo hello >> file.txt` 会将 `hello` 追加到现有的 `file.txt`。
- **Glob 展开：** `*` 展开所有选项。
  - `echo *.py` 会回显所有以 `.py` 结尾的文件名
  - `echo **/*.py` 会回显此目录及所有子目录中以 `.py` 结尾的所有文件名
  - `echo data[0-9].csv` 会回显 `data` 后面有一个数字且以 `.csv` 结尾的所有文件名

更多信息在 [`deno_task_shell` 文档](https://deno.land/manual@v1.35.0/tools/task_runner#task-runner)。