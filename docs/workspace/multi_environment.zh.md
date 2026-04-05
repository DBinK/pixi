# 多环境

### 激励示例

多环境在多种场景下很有用。

- **测试多个包版本**，例如 `py39` 和 `py310` 或 polars `0.12` 和 `0.13`。
- **更小的单一工具环境**，例如 `lint` 或 `docs`。
- **大型开发者环境**，结合所有较小的环境，例如 `dev`。
- **环境的严格超集**，例如 `prod` 和 `test-prod`，其中 `test-prod` 是 `prod` 的严格超集。
- **一个 workspace 中的多个系统要求**，例如 `cuda` 环境和 `cpu` 环境。
- **以及更多。**（欢迎编辑我们的 GitHub 文档添加你的用例。）

这为多用例、多开发者和不同 CI 需求的大型 workspace 准备好了 `pixi`。

## 设计考虑

在设计中我们希望记住以下几点：

1. **用户友好性**：Pixi 是一个面向用户的工具，该功能应从一开始就具有好的错误报告和有用的文档。
2. **保持简单**：不理解多环境功能不应限制用户使用 pixi。该功能对于非多环境用例应该是"不可见"的。
3. **避免自动组合**：为确保依赖解析过程保持可管理，解决方案应避免依赖集的情感爆炸。通过使环境由用户定义而不是通过测试特征矩阵自动推断来实现。
4. **单一环境激活**：设计应允许任何给定时间只有一个环境处于活动状态，简化解析过程并防止冲突。
5. **固定锁文件**：保持固定锁文件的一致性和可预测性至关重要。解决方案必须不仅为作者而且为最终用户提供可靠性，特别是在锁文件创建时。

### Feature 和环境集定义

将环境集引入 `pixi.toml`，这描述了基于 Feature 的环境。将 Feature 引入 `pixi.toml`，可以描述环境的部分。
由于环境不仅仅包含 `dependencies`，可以通过包含以下字段来描述 Feature 字段：

- `dependencies`：conda 包依赖
- `pypi-dependencies`：pypi 包依赖
- `system-requirements`：环境系统要求
- `activation`：环境的激活信息
- `platforms`：环境可以运行的平台。
- `channels`：用于创建环境的 channel。添加 `priority` 字段以允许 channel 连接而不是覆盖。
- `target`：所有上述内容，但也按目标分隔。
- `tasks`：特定 Feature 的任务，一个环境中的任务被选为该环境的默认任务。

```toml title="Default features"
[dependencies] # short for [feature.default.dependencies]
python = "*"
numpy = "==2.3"

[pypi-dependencies] # short for [feature.default.pypi-dependencies]
pandas = "*"

[system-requirements] # short for [feature.default.system-requirements]
libc = "2.33"

[activation] # short for [feature.default.activation]
scripts = ["activate.sh"]
```

```toml title="Different dependencies per feature"
[feature.py39.dependencies]
python = "~=3.9.0"
[feature.py310.dependencies]
python = "~=3.10.0"
[feature.test.dependencies]
pytest = "*"
```

```toml title="Full set of environment modification in one feature"
[feature.cuda]
dependencies = {cuda = "x.y.z", cudnn = "12.0"}
pypi-dependencies = {torch = "1.9.0"}
platforms = ["linux-64", "osx-arm64"]
activation = {scripts = ["cuda_activation.sh"]}
system-requirements = {cuda = "12"}
# Channels concatenate using a priority instead of overwrite, so the default channels are still used.
# Using the priority the concatenation is controlled, default is 0, the default channels are used last.
# Highest priority comes first.
channels = ["nvidia", {channel = "pytorch", priority = -1}] # Results in:  ["nvidia", "conda-forge", "pytorch"] when the default is `conda-forge`
tasks = { warmup = "python warmup.py" }
target.osx-arm64 = {dependencies = {mlx = "x.y.z"}}
```

```toml title="Define tasks as defaults of an environment"
[feature.test.tasks]
test = "pytest"

[environments]
test = ["test"]

# `pixi run test` == `pixi run --environment test test`
```

环境定义应包含以下字段：

- `features: Vec<Feature>`：包含在环境集中的 Feature，也是环境中的默认字段。
- `solve-group: String`：用于在求解阶段将环境分组在一起。
  这对于需要具有相同依赖但可能扩展额外依赖的环境很有用。
  例如，使用额外测试依赖测试生产环境。

```toml title="Creating environments from features"
[environments]
# implicit: default = ["default"]
default = ["py39"] # implicit: default = ["py39", "default"]
py310 = ["py310"] # implicit: py310 = ["py310", "default"]
test = ["test"] # implicit: test = ["test", "default"]
test39 = ["test", "py39"] # implicit: test39 = ["test", "py39", "default"]
```

```toml title="Testing a production environment with additional dependencies"
[environments]
# Creating a `prod` environment which is the minimal set of dependencies used for production.
prod = {features = ["py39"], solve-group = "prod"}
# Creating a `test_prod` environment which is the `prod` environment plus the `test` feature.
test_prod = {features = ["py39", "test"], solve-group = "prod"}
# Using the `solve-group` to solve the `prod` and `test_prod` environments together
# Which makes sure the tested environment has the same version of the dependencies as the production environment.
```

```toml title="Creating environments without including the default feature"
[dependencies]
python = "*"
numpy = "*"

[feature.lint.dependencies]
pre-commit = "*"

[environments]
# Create a custom environment which only has the `lint` feature (numpy isn't part of that env).
lint = {features = ["lint"], no-default-feature = true}
```

### 锁文件结构

在 `pixi.lock` 文件中，一个包现在可以包含一个额外的 `environments` 字段，指定它属于哪个环境。
为避免重复包的 `environments` 字段可以包含多个环境，因此锁文件是最小化的。

```yaml
- platform: linux-64
  name: pre-commit
  version: 3.3.3
  category: main
  environments:
    - dev
    - test
    - lint
  ...:
- platform: linux-64
  name: python
  version: 3.9.3
  category: main
  environments:
    - dev
    - test
    - lint
    - py39
    - default
  ...:
```

### 用户界面环境激活

用户可以通过命令行或配置手动激活所需的环境。
这种方法通过只允许一组 Feature 在任何给定时间处于活动状态来保证无冲突的环境。
对于用户，cli 如下所示：

```shell title="Default behavior"
➜ pixi run python
# Runs python in the `default` environment
```

```shell title="Activating an specific environment"
➜ pixi run -e test pytest
➜ pixi run --environment test pytest
# Runs `pytest` in the `test` environment
```

```shell title="Activating a shell in an environment"
➜ pixi shell -e cuda
pixi shell --environment cuda
# Starts a shell in the `cuda` environment
```

```shell title="Running any command in an environment"
➜ pixi run -e test any_command
# Runs any_command in the `test` environment which doesn't require to be predefined as a task.
```
### 模糊环境选择

可以在多个环境中定义任务，在这种情况下，应提示用户选择环境。

这是一个仅包含任务清单的简单示例：

```toml title="pixi.toml"
[workspace]
name = "test_ambiguous_env"
channels = []
platforms = ["linux-64", "win-64", "osx-64", "osx-arm64"]

[tasks]
default = "echo Default"
ambi = "echo Ambi::Default"
[feature.test.tasks]
test = "echo Test"
ambi = "echo Ambi::Test"

[feature.dev.tasks]
dev = "echo Dev"
ambi = "echo Ambi::Dev"

[environments]
default = ["test", "dev"]
test = ["test"]
dev = ["dev"]
```
运行 `ambi` 任务将提示用户选择环境。
因为它在所有环境中都可用。

```shell title="Interactive selection of environments if task is in multiple environments"
➜ pixi run ambi
? The task 'ambi' can be run in multiple environments.

Please select an environment to run the task in: ›
❯ default # selecting default
  test
  dev

✨ Pixi task (ambi in default): echo Ambi::Test
Ambi::Test
```

如你所见，它运行在 `feature.task` 中定义的任务，但在 `default` 环境中运行。
这是因为 `ambi` 任务在 `test` Feature 中定义，并在 default 环境中被覆盖。
所以 `tasks.default` 现在从任何环境都无法访问。

在此示例中运行的一些其他结果：
```shell
➜ pixi run --environment test ambi
✨ Pixi task (ambi in test): echo Ambi::Test
Ambi::Test

➜ pixi run --environment dev ambi
✨ Pixi task (ambi in dev): echo Ambi::Dev
Ambi::Dev

# dev is run in the default environment
➜ pixi run dev
✨ Pixi task (dev in default): echo Dev
Dev

# dev is run in the dev environment
➜ pixi run -e dev dev
✨ Pixi task (dev in dev): echo Dev
Dev
```

## 初始提案

初始提案：[GitHub Gist by 0xbe7a](https://gist.github.com/0xbe7a/bbf8a323409be466fe1ad77aa6dd5428)

## 实际用例示例

??? tip "Polarify test setup"

    在 `polarify` 中，他们希望结合多个版本测试多个版本的 polars。
    目前这是通过在 GitHub actions 中使用矩阵完成的。
    这可以通过使用多个环境来替换。

    ```toml title="pixi.toml"
    [workspace]
    name = "polarify"
    # ...
    channels = ["conda-forge"]
    platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

    [tasks]
    postinstall = "pip install --no-build-isolation --no-deps --disable-pip-version-check -e ."

    [dependencies]
    python = ">=3.9"
    pip = "*"
    polars = ">=0.14.24,<0.21"

    [feature.py39.dependencies]
    python = "3.9.*"
    [feature.py310.dependencies]
    python = "3.10.*"
    [feature.py311.dependencies]
    python = "3.11.*"
    [feature.py312.dependencies]
    python = "3.12.*"
    [feature.pl017.dependencies]
    polars = "0.17.*"
    [feature.pl018.dependencies]
    polars = "0.18.*"
    [feature.pl019.dependencies]
    polars = "0.19.*"
    [feature.pl020.dependencies]
    polars = "0.20.*"

    [feature.test.dependencies]
    pytest = "*"
    pytest-md = "*"
    pytest-emoji = "*"
    hypothesis = "*"
    [feature.test.tasks]
    test = "pytest"

    [feature.lint.dependencies]
    pre-commit = "*"
    [feature.lint.tasks]
    lint = "pre-commit run --all"

    [environments]
    pl017 = ["pl017", "py39", "test"]
    pl018 = ["pl018", "py39", "test"]
    pl019 = ["pl019", "py39", "test"]
    pl020 = ["pl020", "py39", "test"]
    py39 = ["py39", "test"]
    py310 = ["py310", "test"]
    py311 = ["py311", "test"]
    py312 = ["py312", "test"]
    ```

    ```yaml title=".github/workflows/test.yml"
    jobs:
      tests-per-env:
        runs-on: ubuntu-latest
        strategy:
          matrix:
            environment: [py311, py312]
        steps:
        - uses: actions/checkout@v4
          - uses: prefix-dev/setup-pixi@v0.5.1
            with:
              environments: ${{ matrix.environment }}
          - name: Run tasks
            run: |
              pixi run --environment ${{ matrix.environment }} test
      tests-with-multiple-envs:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
        - uses: prefix-dev/setup-pixi@v0.5.1
          with:
           environments: pl017 pl018
        - run: |
            pixi run -e pl017 test
            pixi run -e pl018 test
    ```

??? tip "Test vs Production example"

    这是一个具有 `test` Feature 和 `prod` 环境的 workspace 示例。
    `prod` 环境是包含运行依赖的生产环境。
    `test` Feature 是一组我们想要放在先前求解的 `prod` 环境之上的依赖和任务。
    这是一个常见的用例，我们想用额外的依赖测试生产环境。

    ```toml title="pixi.toml"
    [workspace]
    name = "my-app"
    # ...
    channels = ["conda-forge"]
    platforms = ["osx-arm64", "linux-64"]

    [tasks]
    postinstall-e = "pip install --no-build-isolation --no-deps --disable-pip-version-check -e ."
    postinstall = "pip install --no-build-isolation --no-deps --disable-pip-version-check ."
    dev = "uvicorn my_app.app:main --reload"
    serve = "uvicorn my_app.app:main"

    [dependencies]
    python = ">=3.12"
    pip = "*"
    pydantic = ">=2"
    fastapi = ">=0.105.0"
    sqlalchemy = ">=2,<3"
    uvicorn = "*"
    aiofiles = "*"

    [feature.test.dependencies]
    pytest = "*"
    pytest-md = "*"
    pytest-asyncio = "*"
    [feature.test.tasks]
    test = "pytest --md=report.md"

    [environments]
    # both default and prod will have exactly the same dependency versions when they share a dependency
    default = {features = ["test"], solve-group = "prod-group"}
    prod = {features = [], solve-group = "prod-group"}
    ```
    在 ci 中你会运行以下命令：
    ```shell
    pixi run postinstall-e && pixi run test
    ```
    在本地你会运行以下命令：
    ```shell
    pixi run postinstall-e && pixi run dev
    ```

    然后在 Dockerfile 中你会运行以下命令：
    ```dockerfile title="Dockerfile"
    FROM ghcr.io/prefix-dev/pixi:latest # this doesn't exist yet
    WORKDIR /app
    COPY . .
    RUN pixi run --environment prod postinstall
    EXPOSE 8080
    CMD ["/usr/local/bin/pixi", "run", "--environment", "prod", "serve"]
    ```

??? tip "Multiple machines from one workspace"

    这是一个应该可以在支持 `cuda` 和 `mlx` 的机器上执行的 ML workspace 示例。
    它也应该可以在不支持 `cuda` 或 `mlx` 的机器上执行，我们使用 `cpu` Feature 来实现。

    ```toml title="pixi.toml"
    [workspace]
    name = "my-ml-workspace"
    description = "A workspace that does ML stuff"
    authors = ["Your Name <your.name@gmail.com>"]
    channels = ["conda-forge", "pytorch"]
    # All platforms that are supported by the workspace as the features will take the intersection of the platforms defined there.
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [tasks]
    train-model = "python train.py"
    evaluate-model = "python test.py"

    [dependencies]
    python = "3.11.*"
    pytorch = {version = ">=2.0.1", channel = "pytorch"}
    torchvision = {version = ">=0.15", channel = "pytorch"}
    polars = ">=0.20,<0.21"
    matplotlib-base = ">=3.8.2,<3.9"
    ipykernel = ">=6.28.0,<6.29"

    [feature.cuda]
    platforms = ["win-64", "linux-64"]
    channels = ["nvidia", {channel = "pytorch", priority = -1}]
    system-requirements = {cuda = "12.1"}

    [feature.cuda.tasks]
    train-model = "python train.py --cuda"
    evaluate-model = "python test.py --cuda"

    [feature.cuda.dependencies]
    pytorch-cuda = {version = "12.1.*", channel = "pytorch"}

    [feature.mlx]
    platforms = ["osx-arm64"]
    # MLX is only available on macOS >=13.5 (>14.0 is recommended)
    system-requirements = {macos = "13.5"}

    [feature.mlx.tasks]
    train-model = "python train.py --mlx"
    evaluate-model = "python test.py --mlx"

    [feature.mlx.dependencies]
    mlx = ">=0.16.0,<0.17.0"

    [feature.cpu]
    platforms = ["win-64", "linux-64", "osx-64", "osx-arm64"]

    [environments]
    cuda = ["cuda"]
    mlx = ["mlx"]
    default = ["cpu"]
    ```

    ```shell title="Executing on a cuda machine"
    pixi run train-model --environment cuda
    # will execute `python train.py --cuda`
    # fails if not on linux-64 or win-64 with cuda 12.1
    ```

    ```shell title="Executing with mlx"
    pixi run train-model --environment mlx
    # will execute `python train.py --mlx`
    # fails if not on osx-arm64
    ```

    ```shell title="Executing on a machine without cuda or mlx"
    pixi run train-model
    ```