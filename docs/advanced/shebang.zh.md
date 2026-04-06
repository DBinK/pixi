!!!warning "仅限类 Unix 系统"
    以下方法仅适用于类 Unix 系统（即 Linux 和 macOS），因为 Windows 不支持 shebang 行。

对于简单脚本，你可以使用 [`pixi exec`](../reference/cli/pixi/exec.md) 直接运行它们，而无需考虑安装依赖或设置环境。
这可以通过在脚本顶部添加 [shebang 行](https://en.wikipedia.org/wiki/Shebang_(Unix)) 来完成，它告诉系统如何执行脚本。
通常，shebang 行以 `#!/usr/bin/env` 开头，后跟要使用的解释器的名称。

除了添加解释器，你也可以在脚本开头添加 `pixi exec`。
你的脚本的唯一要求是你必须在系统上安装 `pixi`。

!!!tip "使脚本可执行"
    你可能需要通过运行 `chmod +x my-script.sh` 使脚本可执行。

```bash title="use-bat.sh"
#!/usr/bin/env -S pixi exec --spec bat -- bash -e

bat my-file.json
```

!!!info "解释发生了什么"
    `#!` 是告诉你的系统如何执行脚本的魔法字符（取 `#!` 之后的所有内容并附加文件名）。
    
    `/usr/bin/env` 用于在系统的 `PATH` 中找到 `pixi`。
    `-S` 选项告诉 `/usr/bin/env` 使用第一个参数作为解释器，其余作为解释器的参数。
    `pixi exec --spec bat` 创建一个只包含 [`bat`](https://github.com/sharkdp/bat) 的临时环境。
    `bash -e`（用 `--` 分隔）是在此环境中执行的命令。
    因此，总的来说，当你运行 `./use-bat.sh` 时，实际上执行的是 `pixi exec --spec bat -- bash -e use-bat.sh`。

你也可以编写包含其依赖的独立 Python 文件。
这个示例展示了一个非常简单的 CLI，它使用 [`py-rattler`](https://conda.github.io/rattler/py-rattler) 和 [`typer`](https://typer.tiangolo.com) 将 Pixi 环境安装到任意前缀。

```python title="install-pixi-environment-to-prefix.py"
#!/usr/bin/env -S pixi exec --spec py-rattler>=0.10.0,<0.11 --spec typer>=0.15.0,<0.16 -- python

import asyncio
from pathlib import Path
from typing import get_args

from rattler import install as rattler_install
from rattler import LockFile, Platform
from rattler.platform.platform import PlatformLiteral
from rattler.networking import Client, MirrorMiddleware, AuthenticationMiddleware
import typer


app = typer.Typer()


async def _install(
    lock_file_path: Path,
    environment_name: str,
    platform: Platform,
    target_prefix: Path,
) -> None:
    lock_file = LockFile.from_path(lock_file_path)
    environment = lock_file.environment(environment_name)
    if environment is None:
        raise ValueError(f"Environment {environment_name} not found in lock file {lock_file_path}")
    records = environment.conda_repodata_records_for_platform(platform)
    if not records:
        raise ValueError(f"No records found for platform {platform} in lock file {lock_file_path}")
    await rattler_install(
        records=records,
        target_prefix=target_prefix,
        client=Client(
            middlewares=[
                MirrorMiddleware({"https://conda.anaconda.org/conda-forge": ["https://repo.prefix.dev/conda-forge"]}),
                AuthenticationMiddleware(),
            ]
        ),
    )


@app.command()
def install(
    lock_file_path: Path = Path("pixi.lock").absolute(),
    environment_name: str = "default",
    platform: str = str(Platform.current()),
    target_prefix: Path = Path("env").absolute(),
) -> None:
    """
    Installs a pixi.lock file to a custom prefix.
    """
    if platform not in get_args(PlatformLiteral):
        raise ValueError(f"Invalid platform {platform}. Must be one of {get_args(PlatformLiteral)}")
    asyncio.run(
        _install(
            lock_file_path=lock_file_path,
            environment_name=environment_name,
            platform=Platform(platform),
            target_prefix=target_prefix,
        )
    )


if __name__ == "__main__":
    app()
```
