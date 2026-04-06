![pixi-browse demo](https://raw.githubusercontent.com/pavelzw/pixi-browse/refs/heads/main/.github/assets/demo-light.gif#only-light)
![pixi-browse demo](https://raw.githubusercontent.com/pavelzw/pixi-browse/refs/heads/main/.github/assets/demo-dark.gif#only-dark)

[pixi-browse](https://github.com/pavelzw/pixi-browse) 是一个用于浏览 conda 包元数据的交互式终端 UI。从任何 conda 通道探索包、版本、依赖项等，直接从终端。

## 功能

- 从任何 conda 通道浏览包（conda-forge、prefix.dev 等）
- **模糊搜索**快速筛选数千个包
- **检查版本**，按平台分组，带有可折叠部分
- **查看详细元数据**，包括依赖项、许可证、校验和、构建信息和时间戳
- **检查包内容** — 直接从工件中提取的文件列表和 `about.json`
- **可点击链接**指向源代码仓库、维护者 GitHub 配置文件和来源提交
- **直接将工件下载**到你的工作目录
- **Vim 风格按键绑定**用于快速键盘导航

## 安装

```bash
pixi global install pixi-browse
```

或无需安装即可使用：

```bash
pixi exec pixi-browse
```

## 用法

```bash
# 浏览 conda-forge（默认）
pixi browse

# 浏览不同的通道
pixi browse -c prefix.dev/conda-forge

# 限制为特定平台
pixi browse -p linux-64 -p osx-arm64
```
