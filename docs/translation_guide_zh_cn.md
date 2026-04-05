# Pixi 中文翻译 Style Guide

## 🎯 目标

* 保持与 Pixi / Conda / Cargo 生态一致
* 降低用户理解成本（尤其 CLI / 报错）
* 保证术语稳定、可复用

---

## 🧭 总体策略

### 术语使用规则

* 首次出现：**中文（英文）**
* 后续：**优先使用英文**
* CLI / 配置字段：**始终保留英文**
* 不强行“全中文化”

---

### 何时保留英文

满足任一条件：

* 用户需要在 CLI 输入（如 `task`, `run`, `install`）
* 配置文件字段（如 `dependencies`, `feature`）
* 无准确中文等价（如 `Feature`）


### 英文术语大小写规范

* 默认使用小写（workspace / environment / manifest）
* 在以下情况使用大写：

  * 术语定义
  * 标题
  * 明确作为专有概念强调时
* Feature 始终使用大写（Feature）
---

### 何时使用中文

* 通用概念（如 环境 / 依赖 / 平台）
* 不影响 CLI 对照理解

---

## ✍️ 术语格式规范

### 标准写法

* 工作区（workspace）
* 项目清单（manifest）
* 环境（environment）

---

### Feature 特例（必须遵守）

* 写法：**Feature（功能模块）**
* 后文：只用 `Feature`
* ❌ 禁止：特性

---

### Channel 规范

二选一（全文统一）：

* 推荐：通道（channel）
* 或：直接使用 channel

---

## 🧱 句式与语气

### 推荐风格

* 简洁、工程化
* 避免文学表达
* 优先动词开头

示例：

* ✅ 使用 `pixi install` 安装依赖
* ❌ 你可以通过执行命令来完成依赖的安装

---

### 中英混排规则

* 英文术语前后留空格
* 代码 / 命令使用反引号

示例：

* 在 workspace 中定义 environment
* 运行 `pixi run test`

---

## 💻 CLI 与代码规范

### CLI 命令

* 永远保持英文
* 使用反引号

```
pixi install
pixi run build
```

---

### 配置文件（pixi.toml）

* key 不翻译
* value 不翻译
* 解释用中文

```toml
[dependencies]
python = "3.11"
```

说明：

* `dependencies`：依赖列表

---

## ⚠️ 常见错误（必须避免）

* ❌ Feature → 特性
* ❌ Channel → 频道
* ❌ Trampolines → 蹦床
* ❌ Prefix → 前缀（单独使用）

---

# 📚 Pixi 扩展术语表（推荐终稿）

## 核心概念

| 英文           | 中文               | 使用建议           | 说明        |
| ------------ | ---------------- | -------------- | --------- |
| Workspace    | 工作区（workspace）   | 后续用 workspace  | 项目根       |
| Manifest     | 项目清单（manifest）   | 后续用 manifest   | pixi.toml |
| Environment  | 环境（environment）  | 可中文            | 隔离环境      |
| Dependencies | 依赖（dependencies） | 可中文            | 包依赖       |
| Channel      | 通道（channel）      | 或直接 channel    | 包来源       |
| Platform     | 平台（platform）     | 可中文            | OS+架构     |
| Feature      | Feature（功能模块）    | ⚠️必须保留英文       | 环境组合单元    |
| Task         | 任务（task）         | 可中文            | 命令封装      |
| Lock file    | 锁文件（lock file）   | 后续用 lock file  | pixi.lock |
| Activation   | 激活（activation）   | 后续用 activation | 环境启用      |
| Solver       | 求解器（solver）      | 可中文            | 依赖解析      |

---

## 环境与解析相关

| 英文          | 中文              | 使用建议 | 说明     |
| ----------- | --------------- | ---- | ------ |
| Resolution  | 解析（resolution）  | 可中文  | 依赖求解过程 |
| Constraint  | 约束（constraint）  | 可中文  | 版本限制   |
| Spec        | 规格（spec）        | 可中文  | 依赖声明   |
| Requirement | 需求（requirement） | 可中文  | 包要求    |
| Pinning     | 固定版本（pinning）   | 可中文  | 锁版本    |
| Conflict    | 冲突（conflict）    | 可中文  | 依赖冲突   |

---

## Conda 生态术语

| 英文              | 中文                   | 使用建议  | 说明         |
| --------------- | -------------------- | ----- | ---------- |
| Package         | 包（package）           | 可中文   | conda 包    |
| Variant         | 变体（variant）          | 可中文   | 构建差异       |
| Subdir          | 子平台目录（subdir）        | 推荐中文  | 如 linux-64 |
| Virtual package | 虚拟包（virtual package） | 可中文   | 系统能力       |
| Prefix          | 环境前缀（prefix）         | 必须带限定 | 安装路径       |
| Build string    | 构建字符串（build string）  | 可中文   | 构建标识       |

---

## 执行与任务系统

| 英文          | 中文                 | 使用建议 | 说明     |
| ----------- | ------------------ | ---- | ------ |
| Task runner | 任务运行器（task runner） | 可中文  | 执行任务   |
| Script      | 脚本（script）         | 可中文  | 任务内容   |
| Command     | 命令（command）        | 可中文  | CLI 指令 |
| Hook        | 钩子（hook）           | 可中文  | 生命周期   |

---

## 全局工具

| 英文           | 中文                 | 使用建议   | 说明   |
| ------------ | ------------------ | ------ | ---- |
| Global Tools | 全局工具（global tools） | 可中文    | 全局安装 |
| Trampolines  | trampolines（启动器机制） | ⚠️保留英文 | 命令转发 |

---

## 平台与构建

| 英文                | 中文                      | 使用建议 | 说明    |
| ----------------- | ----------------------- | ---- | ----- |
| Host platform     | 主机平台（host platform）     | 可中文  | 当前系统  |
| Target platform   | 目标平台（target platform）   | 可中文  | 构建目标  |
| Cross-compilation | 交叉编译（cross-compilation） | 可中文  | 跨平台构建 |

---

## 其它重要概念（常被忽略）

| 英文              | 中文                    | 使用建议 | 说明   |
| --------------- | --------------------- | ---- | ---- |
| Reproducibility | 可复现性（reproducibility） | 可中文  | 核心理念 |
| Isolation       | 隔离性（isolation）        | 可中文  | 环境隔离 |
| Deterministic   | 确定性（deterministic）    | 可中文  | 构建稳定 |
| Metadata        | 元数据（metadata）         | 可中文  | 包信息  |

---
