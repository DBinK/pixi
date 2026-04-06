你可以使用 `pixi` 来管理你的 R 依赖。conda-forge 通道包含广泛的 R 包，可以使用 `pixi` 安装。

## 安装 R 包

R 包在 conda-forge 通道中通常以 `r-` 为前缀。要安装 R 包，可以使用以下命令：

```bash
pixi add r-<package-name>
# 例如
pixi add r-ggplot2
```

## 在 RStudio 中使用 R 包

要在 RStudio 中使用 `pixi` 安装的 R 包，你需要从激活的环境中运行 `rstudio`。可以通过从 `pixi shell` 或 `pixi.toml` 文件中的任务运行 RStudio 来实现。

## 完整示例

完整示例可以在这里找到：[RStudio 示例](https://github.com/prefix-dev/pixi/tree/main/examples/r)。
以下是一个设置 RStudio 任务的 `pixi.toml` 文件示例：

```toml
[workspace]
name = "r"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-64", "osx-arm64"]

[target.linux.tasks]
rstudio = "rstudio"

[target.osx.tasks]
rstudio = "open -a rstudio"
# 或使用完整路径：
# rstudio = "/Applications/RStudio.app/Contents/MacOS/RStudio"

[dependencies]
r = ">=4.3,<5"
r-ggplot2 = ">=3.5.0,<3.6"
```

RStudio 加载后，你可以执行使用 `ggplot2` 包的以下 R 代码：

```R
# Load the ggplot2 package
library(ggplot2)

# Load the built-in 'mtcars' dataset
data <- mtcars

# Create a scatterplot of 'mpg' vs 'wt'
ggplot(data, aes(x = wt, y = mpg)) +
  geom_point() +
  labs(x = "Weight (1000 lbs)", y = "Miles per Gallon") +
  ggtitle("Fuel Efficiency vs. Weight")
```

!!! Note
    此示例假设你已系统级安装 RStudio。
    我们也在努力更新 RStudio 以及 Windows 上的 R 解释器构建，以与 `pixi` 实现最大兼容性。
