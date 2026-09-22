# 安装与特性

## Rust 版本

当前 crate 使用：

```rust
#![feature(generic_const_exprs)]
```

因此需要 nightly Rust。这个 feature 主要用于 `ArmKineCache<N>` 这类需要 `[(); N + 1]` 的固定尺寸运动学缓存。

## Cargo 依赖

已发布包的依赖形式：

```toml
[dependencies]
robot_behavior = "0.6"
```

本页描述当前开发源码；仅相同版本号不能证明 crates.io 已发布包包含相同 API。源码对齐开发应固定对应本地/git 版本；图适配另外开启 `features = ["roplat"]`，见 [Roplat 集成](roplat-integration.md)。

Roplat workspace 中通常通过顶层 `[patch.crates-io]` 或 path 依赖复用本地源码。

## Feature

| Feature | 作用 | 备注 |
|---|---|---|
| 默认 | 核心 Rust trait 与工具 | 推荐新驱动优先使用 |
| `roplat` | 启用 `robot_behavior::roplat` 和 roplat 依赖 | 默认关闭 |
| `ffi` | 打开 FFI 模块门控 | 作为外语绑定基础 |
| `to_py` | 启用 PyO3 | 同时启用 `ffi` |
| `to_cxx` | 启用 `cxx` | 同时启用 `ffi` |
| `to_c` | C 接口门控 | 同时启用 `ffi` |

核心行为接口不需要任何额外 feature。

## 常用命令

在 `drives` workspace 中：

```powershell
cargo check -p robot_behavior --no-default-features
cargo check -p robot_behavior --features roplat
cargo test -p robot_behavior --lib
```

生成页面：

```powershell
cd robot_behavior_page
mkdocs build
```

本地预览：

```powershell
mkdocs serve
```

## 依赖角色

主要依赖包括：

- `nalgebra`：SE(3)、向量、矩阵、雅可比。
- `serde` / `serde_json`：状态、位姿、轨迹文件序列化。
- `thiserror` / `anyhow`：错误统一。
- `copp`：`utils::trajectory` 中的时间最优/限位轨迹规划。
- `roplat`：可选节点/节律适配，需启用同名 feature。
- `pyo3` / `cxx`：可选外语绑定。
