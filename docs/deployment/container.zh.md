将 Pixi 包投入生产的一种方法是使用 Docker 或 Podman 等工具进行容器化。

<!-- Keep in sync with https://github.com/prefix-dev/pixi-docker/blob/main/README.md -->

我们在 [`pixi-docker`](https://github.com/prefix-dev/pixi-docker) 提供了一个简单的 Docker 镜像，其中包含在不同基础镜像之上的 Pixi 可执行文件。

镜像可在 [ghcr.io/prefix-dev/pixi](https://ghcr.io/prefix-dev/pixi) 获取。

有不同的标签可用于不同的基础镜像：

- `latest` - 基于 `ubuntu:jammy`
- `focal` - 基于 `ubuntu:focal`
- `bullseye` - 基于 `debian:bullseye`
- `noble-cuda-12.9.1` - 基于 `nvidia/cuda:12.9.1-base-ubuntu24.04`
- `noble-cuda-13.0.0` - 基于 `nvidia/cuda:13.0.0-base-ubuntu24.04`
- ... 等等

!!!tip "所有标签"
    有关所有标签，请查看[构建脚本](https://github.com/prefix-dev/pixi-docker/blob/main/.github/workflows/build.yml)。

!!!tip "Docker 与 pixi 最佳实践"
    [@pavelzw](https://github.com/pavelzw) 写了一篇关于[使用 pixi 将 conda 环境运送到生产的博客文章](https://tech.quantco.com/blog/pixi-production)。
    如果你想了解更多关于在 Docker 中使用 pixi 的最佳实践，请查看他们的博客文章。

### 使用示例

以下示例使用 Pixi Docker 镜像作为多阶段构建的基础镜像。
它还利用了 `pixi shell-hook`，以不依赖在生产容器中安装 Pixi。

!!!tip "更多示例"
    更多示例请查看 [pavelzw/pixi-docker-example](https://github.com/pavelzw/pixi-docker-example)。

```Dockerfile
FROM ghcr.io/prefix-dev/pixi:0.41.4 AS build

# 将源代码、pixi.toml 和 pixi.lock 复制到容器
WORKDIR /app
COPY . .
# 安装依赖到 `/app/.pixi/envs/prod`
# 使用 `--locked` 确保 lockfile 与 pixi.toml 保持同步
RUN pixi install --locked -e prod
# 创建 shell-hook bash 脚本来激活环境
RUN pixi shell-hook -e prod -s bash > /shell-hook
RUN echo "#!/bin/bash" > /app/entrypoint.sh
RUN cat /shell-hook >> /app/entrypoint.sh
# 扩展 shell-hook 脚本以运行传递给容器的命令
RUN echo 'exec "$@"' >> /app/entrypoint.sh

FROM ubuntu:24.04 AS production
WORKDIR /app
# 只将生产环境复制到 prod 容器
# 请注意，"prefix"（路径）需要与构建容器中的相同
COPY --from=build /app/.pixi/envs/prod /app/.pixi/envs/prod
COPY --from=build --chmod=0755 /app/entrypoint.sh /app/entrypoint.sh
# 将你的项目代码也复制到容器
COPY ./my_project /app/my_project

EXPOSE 8000
ENTRYPOINT [ "/app/entrypoint.sh" ]
# 在 pixi 环境中运行你的应用
CMD [ "uvicorn", "my_project:app", "--host", "0.0.0.0" ]
```
