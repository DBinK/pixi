# 蹦床（Trampolines）

为了提高效率，`pixi` 使用**蹦床**——小型专用二进制文件，用于在执行主二进制文件之前管理配置和环境设置。
蹦床方法允许跳过执行具有显著性能影响的激活脚本。

当你执行全局安装的可执行文件时，蹦床执行以下步骤序列：

* 每个蹦床首先读取一个以正在执行的二进制文件命名的配置文件。此配置文件为 JSON 格式（例如 `python.json`），包含有关如何设置环境的关键信息。配置文件存储在 [`$PIXI_HOME`](../reference/environment_variables.md)`/bin/trampoline_configuration` 中。
* 一旦加载了配置并设置了环境，蹦床就会用正确的环境设置执行原始二进制文件。
* 安装新二进制文件时，一个新的蹦床被放置在 [`$PIXI_HOME`](../reference/environment_variables.md)`/bin` 目录中，并硬链接到 [`$PIXI_HOME`](../reference/environment_variables.md)`/bin/trampoline_configuration/trampoline_bin`。这优化了存储空间，避免了相同蹦床的重复。

蹦床将负责确保 `PATH` 包含你本地 `PATH` 上的最新更改，同时避免在安装期间缓存临时的 `PATH` 更改。
如果你想控制 pixi 考虑的基础 `PATH`，你可以在 shell 启动脚本中设置 `export PIXI_BASE_PATH=$PATH`。