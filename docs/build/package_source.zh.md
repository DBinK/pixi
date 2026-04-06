# 包源码

默认情况下，包定义假定源码位置在包定义的根目录中。

例如，如果你的包有以下结构：
```text
my_package
├── pixi.toml
├── src
│   └── my_code.cpp
└── include
    └── my_code.h
```
构建后端应该有不从哪里获取源代码的合理默认值。
除了 `pixi-build-rattler-build` 后端你在 `recipe.yaml` 中指定源码外，构建后端将默认为包清单所在的目录

或者，你可以指定源码的位置。

## 路径
如果你的源码位于其他地方，你可以使用 `package.build.source.path` 字段指定源码的位置。

例如，如果你的包有以下结构：
```text
my_package
├── pixi.toml
└── source
    ├── src
    │   └── my_code.cpp
    └── include
        └── my_code.h
```
你可以像这样指定源码的位置：
```toml
[package.build.source]
path = "source"
```

这也可以与相对路径一起使用：
```toml
[package.build.source]
path = "../my_other_source_directory"
```

这可以与 git 子模块很好地结合使用。

## Git
如果你的源码托管在 git 仓库中，你可以使用 `package.build.source.git` 字段指定仓库 URL。

```toml
[package.build.source]
git = "https://github.com/user/repo.git"
```

你可以固定到特定的分支、标签或提交：
```toml
[package.build.source]
git = "https://github.com/user/repo.git"
branch = "main"
```

```toml
[package.build.source]
git = "https://github.com/user/repo.git"
tag = "v1.0.0"
```

```toml
[package.build.source]
git = "https://github.com/user/repo.git"
rev = "abc123"
```

如果源码位于仓库的子目录中，你也可以指定：
```toml
[package.build.source]
git = "https://github.com/user/repo.git"
tag = "v1.0.0"
subdirectory = "packages/mypackage"
```