`pixi info` 打印出有用的信息，用于调试情况或获取你的机器/工作区的概览。
这些信息也可以使用 `--json` 标志以 `json` 格式获取，这对以编程方式读取它很有用。

```title="Running pixi info in the pixi repo"
➜ pixi info
      Pixi version: 0.13.0
          Platform: linux-64
  Virtual packages: __unix=0=0
                  : __linux=6.5.12=0
                  : __glibc=2.36=0
                  : __cuda=12.3=0
                  : __archspec=1=x86_64
         Cache dir: /home/user/.cache/rattler/cache
      Auth storage: /home/user/.rattler/credentials.json

Workspace
------------
           Version: 0.13.0
     Manifest file: /home/user/development/pixi/pixi.toml
      Last updated: 25-01-2024 10:29:08

Environments
------------
default
          Features: default
          Channels: conda-forge
  Dependency count: 10
      Dependencies: pre-commit, rust, openssl, pkg-config, git, mkdocs, mkdocs-material, pillow, cairosvg, compilers
  Target platforms: linux-64, osx-arm64, win-64, osx-64
             Tasks: docs, test-all, test, build, lint, install, build-docs
```

## 全局信息

信息输出的第一部分始终可用，告诉你 Pixi 在你的机器上能读取到什么。

### 平台

这定义了 pixi 认为你当前所在的平台。
如果这不正确，请在 [Pixi repo](https://github.com/prefix-dev/pixi) 上提交问题。

### 虚拟包

Pixi 能在你的机器上找到的虚拟包。

在 Conda 生态系统中，你可以依赖虚拟包。
这些包不是将被安装的真实依赖，而是在求解步骤中用来查找包是否可以安装在机器上的依赖。
一个简单的例子：当一个包依赖于主机上存在 CUDA 驱动程序时，它可以通过依赖 `__cuda` 虚拟包来做到这一点。
在这种情况下，如果 Pixi 在你的机器上找不到 `__cuda` 虚拟包，安装将失败。

### 缓存目录

Pixi 存储其缓存的目录。
查看[缓存文档](../workspace/environment.md#caching-packages)获取更多信息。

### 认证存储

查看[认证文档](../deployment/authentication.md)

### 缓存大小

[需要 `--extended`]

前面提到的"缓存目录"的大小，以 MiB 为单位。

## 工作区信息

`Workspace` 下面的信息是关于你当前所在的工作区。
只有当你的路径有 [manifest 文件](../reference/pixi_manifest.md) 时，这些信息才可用。

### Manifest 文件

描述工作区的 [manifest 文件](../reference/pixi_manifest.md) 的路径。

### 最后更新

lock file 最后一次更新的时间，无论是手动还是由 Pixi 本身更新。

## 环境信息

每个环境定义的环境信息。如果你没有任何环境定义，这只会显示 `default` 环境。

### Features

这列出了环境中启用的功能。对于默认，这只是 `default`。

### 通道

此环境中使用的通道列表。

### 依赖项数量

为此环境定义的依赖项数量（不是已安装的依赖项数量）。

### 依赖项

为此环境定义的依赖项列表。

### 目标平台

工作区定义的平台。
