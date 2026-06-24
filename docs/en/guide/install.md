# Installation and Features

## Rust Requirement

The source currently uses:

```rust
#![feature(generic_const_exprs)]
#![allow(incomplete_features)]
```

Nightly Rust is therefore required. The main use case is fixed-size caches such as `ArmKineCache<N>`, which stores `N + 1` link frames.

## Cargo Dependency

```toml
[dependencies]
robot_behavior = "0.6"
```

Inside the Roplat workspace, the crate is usually consumed through a top-level `[patch.crates-io]` entry or a path dependency to the local source.

## Features

| Feature | Purpose | Notes |
|---|---|---|
| `default` | empty | the core Rust traits need no extra feature |
| `ffi` | enables the FFI module gate | base gate |
| `to_py` | `ffi` + `pyo3` | Python binding support |
| `to_cxx` | `ffi` + `cxx` | C++ binding support |
| `to_c` | `ffi` | C-facing gate |

!!! warning
    FFI macros and examples are still migrating. The main documentation follows the current Rust trait API. Before relying on FFI, run `cargo check --features to_py` or the corresponding target feature for your driver.

## Verification Commands

From the `drives` workspace:

```powershell
cargo build -p robot_behavior
cargo test -p robot_behavior --doc
```

From a standalone `robot_behavior` checkout:

```powershell
cargo build
cargo test --doc
```
