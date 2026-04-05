# 锁文件

> 锁文件是环境的保护者，而 Pixi 是解锁它的钥匙。

## 什么是锁文件？

要回答这个问题，我们首先需要突出 manifest 和锁文件之间的区别。

Manifest 列出你的项目的直接依赖。
当你安装环境时，此 manifest 经历"依赖解析"：找到你请求的依赖的所有依赖，等等，直到最底层。
在解析过程中，确保解析的版本彼此兼容。

锁文件列出了在解析过程中解析的确切依赖项——包、它们的版本以及用于包管理的有用元数据。

锁文件通过一个相对较小的文件提高可复现性。
安装程序可以创建完全相同的环境，而无需管理实际包内容本身。

!!! 警告"不要编辑锁文件"
    锁文件是为机器设计的，为了便于检查而让人可读。它不应该手动编辑。

## Pixi 中的锁文件

Pixi——像许多其他现代包管理器一样——原生支持锁文件。该文件名为 `pixi.lock`。

在创建锁文件期间，Pixi 解析包——针对 manifest 中列出的所有环境和平台。
这大大增加了项目的可复现性，使其在不同的操作系统或 CPU 架构上易于使用——事实上，在很多情况下，共享锁文件可以代替共享 Docker 容器！这使得在 CI 中重建本地开发环境非常容易。

Pixi 锁文件也是人类可读的，所以你可以查看列出了哪些包，而无需额外工具——以及轻松跟踪文件的变化（不要编辑它——你读了上面的警告块，对吧？）。

## 锁文件更改

许多 Pixi 命令会在锁文件不存在时创建它，或者在需要时更新它。例如，当你安装包时，Pixi 经历以下过程：

1. 用户请求安装一个包
2. 依赖解析
3. 生成和写入锁文件
4. 安装结果包

此外，Pixi 确保锁文件与你的 manifest 以及已安装的环境保持同步。
如果检测到它们不同步，它将重新生成锁文件。你可以在[锁文件满足性](#锁文件满足性)部分阅读更多相关信息。

以下命令将检查并在需要时自动更新锁文件：

- [`pixi install`](../reference/cli/pixi/install.md)
- [`pixi run`](../reference/cli/pixi/run.md)
- [`pixi shell`](../reference/cli/pixi/shell.md)
- [`pixi shell-hook`](../reference/cli/pixi/shell-hook.md)
- [`pixi tree`](../reference/cli/pixi/tree.md)
- [`pixi list`](../reference/cli/pixi/list.md)
- [`pixi add`](../reference/cli/pixi/add.md)
- [`pixi remove`](../reference/cli/pixi/remove.md)

如果想删除锁文件，你可以简单地删除它——当运行上述命令之一时，它将使用最新包版本重新生成。

你可能希望更好地控制 manifest、锁文件和创建环境之间的交互。有额外的命令行选项可以帮助这一点：

- `--frozen`：按锁文件中定义安装环境，如果 `pixi.lock` 与 [manifest 文件](../reference/pixi_manifest.md) 不同步则不更新 `pixi.lock`。也可以通过 `PIXI_FROZEN` 环境变量控制（例如：`PIXI_FROZEN=true`）。
- `--locked`：仅在 `pixi.lock` 与 [manifest 文件](../reference/pixi_manifest.md) 同步时才安装。也可以通过 `PIXI_LOCKED` 环境变量控制（例如：`PIXI_LOCKED=true`）。与 `--frozen` 冲突。

## 提交你的锁文件

可复现性在许多项目中非常重要（例如，部署软件服务、研究项目、数据分析）。
环境的可复现性有助于结果的可复现性——它确保你的开发者和部署机器都使用相同的包。

犹豫是否提交锁文件？考虑以下几点：

- 用于可复现环境的 Docker 镜像**总是更大**。
- Git 可以很好地处理 YAML。
- 它作为依赖解析的缓存，提供**更快的安装和 CI**。
- 你不需要它……直到你需要。在压力下删除或忽略比重新创建一个更容易。

但是，有一类项目不能简单地提交锁文件——还有额外的考虑因素。
也就是说，这是当开发**库**时。

---

库具有不断发展的性质，需要针对广泛范围的包版本测试环境以确保兼容性。
这包括一个包含包最新可用版本的环境。

### 库的额外考虑

如果你在库项目中提交锁文件，你可能还需要考虑以下几点：

- **升级锁文件：** 你希望多久升级一次开发者使用的锁文件？你想在主仓库历史中做这些升级吗？你想通过（例如）[Renovate Bot](https://docs.renovatebot.com/modules/manager/pixi/) 或自定义 CI 作业管理这个锁文件吗？
- **针对最新版本测试的自定义 CI 工作流：** 你想要一个针对最新依赖版本测试的工作流吗？如果是——你可能希望在 cron 计划上运行以下 CI 工作流：
  - 在运行 `setup-pixi` 操作之前删除 `pixi.lock`
  - 运行你的测试
  - 如果测试失败：
    - 通过使用 `pixi-diff` 和 `pixi-diff-to-markdown` 查看生成的 `pixi.lock` 与 `main` 中的有何不同
    - 自动提出一个问题，以便在项目仓库中跟踪

你可以看到上述考虑因素是如何被以下项目探索的：
- Scipy（正在探索——一旦可用将更新 PR 链接。[Issue](https://github.com/scipy/scipy/issues/23637))

---

如果你最终决定不提交锁文件，放弃它的好处，你可能想使用诸如 [Parcels-code/pixi-lock](https://github.com/parcels-code/pixi-lock) 之类的操作来缓存生成的锁文件并加速你的 CI。

### 文件结构

Pixi 锁文件分为两部分。

- workspace 中使用的环境——列出包含的包。例如：

```yaml
environments:
    default:
        channels:
          - url: https://conda.anaconda.org/conda-forge/
        packages:
            linux-64:
            ...
            - conda: https://conda.anaconda.org/conda-forge/linux-64/python-3.12.2-hab00c5b_0_cpython.conda
            ...
            osx-64:
            ...
            - conda: https://conda.anaconda.org/conda-forge/osx-64/python-3.12.2-h9f0c242_0_cpython.conda
            ...
```

- 包本身的定义。例如：

```yaml
- kind: conda
  name: python
  version: 3.12.2
  build: h9f0c242_0_cpython
  subdir: osx-64
  url: https://conda.anaconda.org/conda-forge/osx-64/python-3.12.2-h9f0c242_0_cpython.conda
  sha256: 7647ac06c3798a182a4bcb1ff58864f1ef81eb3acea6971295304c23e43252fb
  md5: 0179b8007ba008cf5bec11f3b3853902
  depends:
    - bzip2 >=1.0.8,<2.0a0
    - libexpat >=2.5.0,<3.0a0
    - libffi >=3.4,<4.0a0
    - libsqlite >=3.45.1,<4.0a0
    - libzlib >=1.2.13,<1.3.0a0
    - ncurses >=6.4,<7.0a0
    - openssl >=3.2.1,<4.0a0
    - readline >=8.2,<9.0a0
    - tk >=8.6.13,<8.7.0a0
    - tzdata
    - xz >=5.2.6,<6.0a0
  constrains:
    - python_abi 3.12.* *_cp312
  license: Python-2.0
  size: 14596811
  timestamp: 1708118065292
```

### 锁文件版本

锁文件也有一个版本号，以确保锁文件与本地版本的 `pixi` 兼容。

```yaml
version: 6
```

Pixi 与锁文件向后兼容，但不向前兼容。
这意味着你可以将较旧的锁文件与较新版本的 Pixi 一起使用，但不能反过来。

## 锁文件满足性

锁文件是环境的描述，它应该始终是可满足的。
可满足意味着给定的 manifest 文件和创建的环境与锁文件同步。
如果锁文件不可满足，Pixi 将自动生成新的锁文件。

检查锁文件是否可满足的步骤：

- manifest 文件中的所有 `environments` 都在锁文件中
- manifest 文件中的所有 `channels` 都在锁文件中
- manifest 文件中的所有 `packages` 都在锁文件中，并且锁文件中的版本与 manifest 文件中的要求兼容，无论是 conda 还是 pypi 包。
  - conda 包使用 `matchspec`，可以匹配我们存储在锁文件中的所有信息，甚至 `timestamp`、`subdir` 和 `license`。
- 如果添加了 `pypi-dependencies`，锁文件中所有是 python 包 的 conda 包都有一个 `purls` 字段。
- 所有 pypi 可编辑包的哈希值都是正确的。
- 锁文件中每个包只有一个条目。

如果你想了解更多细节，请查看[实际代码](https://github.com/prefix-dev/pixi/blob/main/crates/pixi_core/src/lock_file/satisfiability/mod.rs)，因为这是实际代码的简化版本。