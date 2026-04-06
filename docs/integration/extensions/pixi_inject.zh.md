[pixi-inject](https://github.com/pavelzw/pixi-inject) 是一个简单的可执行文件，用于将 conda 包注入到现有的 pixi 环境中。

```
pixi inject --environment default --package my-package-0.1.0-py313h8aa417a_0.conda
```

你也可以指定自定义 conda 前缀来注入包。

```
pixi inject --prefix /path/to/conda/env --package my-package-0.1.0-py313h8aa417a_0.conda
```
