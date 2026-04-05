# Conda 和 PyPI

Pixi 构建在 conda 和 PyPI 生态系统之上。

**Conda** 是一个跨平台、跨语言的包生态系统，允许用户安装包和管理环境。
它在数据科学和机器学习社区中广泛使用，但也用于其他领域。
它的力量来自它总是安装二进制包的事实，这意味着它不需要编译任何东西。
这使得生态系统非常快速和易于使用。

**PyPI** 是 Python 包索引，是 Python 包的主要包索引。
它是一个比 conda 大得多的生态系统，尤其是因为上传包的门槛较低。
这意味着有很多可用的包，但也意味着包的质量并不总是像 conda 生态系统那样高。

Pixi 可以从**两个****生态系统**安装包，但它使用 **conda 优先方法**。

简化的过程如下：

1. 解析 conda 依赖。
2. 将 conda 包映射到 PyPI 包。
3. 解析剩余的 PyPI 依赖。

## 工具比较

这是 conda 和 PyPI 生态系统功能的不完全比较。

| 功能 | Conda | PyPI |
| ------- | ----- | ---- |
| 包格式 | 二进制 | 源代码和二进制（wheel） |
| 包管理器 | [`conda`](https://github.com/conda/conda)、[`mamba`](https://github.com/mamba-org/mamba)、[`micromamba`](https://github.com/mamba-org/mamba)、[`pixi`](https://github.com/prefix-dev/pixi)  | [`pip`](https://github.com/pypa/pip)、[`poetry`](https://github.com/python-poetry/poetry)、[`uv`](https://github.com/astral-sh/uv)、[`pdm`](https://github.com/pdm-project/pdm)、[`hatch`](https://github.com/pypa/hatch)、[`rye`](https://github.com/astral-sh/rye)、[`pixi`](https://github.com/prefix-dev/pixi) |
| 环境管理 | [`conda`](https://github.com/conda/conda)、[`mamba`](https://github.com/mamba-org/mamba)、[`micromamba`](https://github.com/mamba-org/mamba)、[`pixi`](https://github.com/prefix-dev/pixi) | [`venv`](https://docs.python.org/3/library/venv.html)、[`virtualenv`](https://virtualenv.pypa.io/en/latest/)、[`pipenv`](https://pipenv.pypa.io/en/latest/)、[`pyenv`](https://github.com/pyenv/pyenv)、[`uv`](https://github.com/astral-sh/uv)、[`poetry`](https://github.com/python-poetry/poetry)、[`pixi`](https://github.com/prefix-dev/pixi) |
| 包构建 | [`conda-build`](https://github.com/conda/conda-build)、[`pixi`](https://github.com/prefix-dev/pixi) | [`setuptools`](https://github.com/pypa/setuptools)、[`poetry`](https://github.com/python-poetry/poetry)、[`flit`](https://github.com/pypa/flit)、[`hatch`](https://github.com/pypa/hatch)、[`uv`](https://github.com/astral-sh/uv)、[`rye`](https://github.com/astral-sh/rye) |
| 包索引 | [`conda-forge`](https://prefix.dev/channels/conda-forge)、[`bioconda`](https://prefix.dev/channels/bioconda)、以及更多 | [pypi.org](https://pypi.org) |

## Astral 的 `uv`

Pixi 使用 [`uv`](https://github.com/astral-sh/uv) 库来处理 PyPI 包。
Pixi 本身不安装 `uv`（工具）：因为两个工具都是用 Rust 构建的，它被用作库。

我们非常感谢 [Astral](https://astral.sh) 团队在 `uv` 上的工作，这是一个很棒的库，允许我们以前更好的方式处理 PyPI 包。

最初，我们在 `pixi` 旁边构建了一个名为 `rip` 的库，与 `uv` 有相同的目标，但我们决定切换到 `uv`，因为它很快成为一个更成熟的库，并且有很多我们需要的功能。

- [最初宣布 `rip` 的博客文章](https://prefix.dev/blog/pypi_support_in_pixi)
- [宣布切换到 `uv` 的博客文章](https://prefix.dev/blog/uv_in_pixi)

## 求解器

因为 Pixi 支持两个生态系统，它目前需要两个不同的求解器来处理依赖。

- [`resolvo`](https://github.com/prefix-dev/resolvo) 库用于求解 conda 依赖。在 [`rattler`](https://github.com/conda/rattler) 中实现。
- [`PubGrub`](https://github.com/pubgrub-rs/pubgrub) 库用于求解 PyPI 依赖。在 [`uv`](https://github.com/astral-sh/uv) 中实现。

!!! 注意
    Pixi 的圣杯是拥有一个可以处理两个生态系统的单一求解器。
    由于 resolvo 编写为支持两个生态系统，它也可以用于 PyPI 包，但尚未实现。

因为 PyPI 包需要一个基础环境来安装到，我们需要在解析之前首先解析 conda 依赖的 conda 优先方法。

Pixi 首先运行 conda（rattler）求解器，它将解析 conda 依赖。
然后它使用 [`parselmouth`](https://github.com/prefix-dev/parselmouth) 将 conda 包映射到 PyPI 包。
然后它运行 PyPI（uv）求解器，它将解析剩余的 PyPI 依赖。

结果是，如果两者都可用并指定为依赖，Pixi 将安装 conda 包（而不是 PyPI 包）。

这是一个这如何在实践中工作的例子：
```toml title="pixi.toml"
[dependencies]
python = ">=3.8"
numpy = ">=1.21.0"

[pypi-dependencies]
numpy = ">=1.21.0"
```

结果如下输出：
```output
➜ pixi list -x
Package  Version  Build               Size      Kind   Source
numpy    2.3.0    py313h41a2e72_0     6.2 MiB   conda  https://conda.anaconda.org/conda-forge/
python   3.13.5   hf3f3da0_102_cp313  12.3 MiB  conda  https://conda.anaconda.org/conda-forge/
```

在这个例子中，Pixi 首先解析 conda 依赖并安装 `numpy` 和 `python` conda 包。
然后它将 `numpy` conda 包映射到 `numpy` PyPI 包并解析任何 PyPI 依赖。
由于没有剩余的 PyPI 依赖（如 `numpy` 已作为 conda 包安装），不会安装 PyPI 包。

另一个例子是当你有一个未指定为 conda 包依赖的 PyPI 包依赖时：
```toml title="pixi.toml"
[dependencies]
python = ">=3.8"

[pypi-dependencies]
numpy = ">=1.21.0"
```
结果如下输出：
```output
> pixi list --explicit
Package  Version  Build               Size      Kind   Source
numpy    2.3.1                        43.8 MiB  pypi   numpy-2.3.1-cp313-cp313-macosx_11_0_arm64.whl
python   3.13.5   hf3f3da0_102_cp313  12.3 MiB  conda  https://conda.anaconda.org/conda-forge/
```
在这个例子中，Pixi 首先解析 conda 依赖并安装 `python` conda 包。
然后，由于 `numpy` 未指定为 conda 依赖，Pixi 解析 PyPI 依赖并安装 `numpy` PyPI 包。

要覆盖或更改 conda 包到 PyPI 包 的映射，你可以使用 `pixi.toml` 文件中的 [`conda-pypi-map`](../reference/pixi_manifest.md#conda-pypi-map-optional) 字段。

### 固定包冲突

当尝试安装 conda 和 PyPI 依赖时，
例如使用以下 `pixi.toml` manifest：

```toml title="pixi.toml"
[dependencies]
typing_extensions = "*"

[pypi-dependencies]
typing_extensions = "==4.14"
```

你可能会遇到以下错误消息：

```
Error:   × failed to solve the pypi requirements of environment 'default' for platform 'osx-arm64'
  ├─▶ failed to resolve pypi dependencies
  ╰─▶ Because you require typing-extensions==4.14 and typing-extensions==4.15.0, we can conclude that your requirements are unsatisfiable.
  help: The following PyPI packages have been pinned by the conda solve, and this version may be causing a conflict:
        typing-extensions==4.15.0
```

这是由于（当前）两阶段求解。
首先，解析 conda 依赖。
由于任何版本（`*`）的 `typing_extensions` 都有效，
它解析为最新可用的版本（本例中为 `4.15.0`）。
然后，解析 PyPI 依赖（`typing_extensions == 4.14`），
并带有已解析的 conda 依赖的约束（`typing_extensions == 4.15.0`）。
这两个依赖不兼容，求解失败。

虽然这可能是一个trivial case，
当包不是显式依赖时就不是很明显了：

```toml title="pixi.toml"
[dependencies]
some-conda-package = "*"  #  depends on any typing-extensions

[pypi-dependencies]
some-pypi-package = "==0.1.0"  # depends on typing-extensions<4.14
```

这会产生以下求解错误：

```
Error:   × failed to solve the pypi requirements of environment 'default' for platform 'osx-arm64'
  ├─▶ failed to resolve pypi dependencies
  ╰─▶ Because some-pypi-dependency==0.1.0 depends on typing-extensions<4.15 and typing-extensions==4.15.0, we can conclude that some-pypi-dependency==0.1.0
      cannot be used.
      And because only some-pypi-dependency==0.1.0 is available and you require some-pypi-dependency, we can conclude that your requirements are unsatisfiable.
  help: The following PyPI packages have been pinned by the conda solve, and this version may be causing a conflict:
        typing-extensions==4.15.0
```

在这种情况下，解决方案是在 conda 依赖中添加 `typing-extensions < 4.15`，
也就是 PyPI 包施加的相同约束：

```toml title="pixi.toml"
[dependencies]
some-conda-package = "*"  #  depends on any typing-extensions
typing_extensions = "<4.15"

[pypi-dependencies]
some-pypi-package = "==0.1.0"  # depends on typing-extensions<4.14
```