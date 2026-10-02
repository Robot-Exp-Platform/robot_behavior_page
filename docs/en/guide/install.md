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
robot_behavior = { version = "0.6.0", git = "ssh://git@github.com/Robot-Exp-Platform/robot_behavior.git", rev = "781245ca4d662a693cfb24955328a5a610859591" }
```

These pages describe the current development source; the version string alone does not prove that a crates.io release contains the same API. For source-aligned graph development, use the matching local/git revision and enable `features = ["roplat"]`. See [Roplat integration](roplat-integration.md).

The prepared versions are robot_behavior 0.6.0 and roplat 0.3.0; neither has been uploaded to crates.io. The published roplat 0.2.2 predates the current execution API and cannot replace the pinned core source. Independent drivers retain complete pinned Git dependency declarations. The integration workspace applies path patches for those exact Git sources at its root. SSH uses the caller's existing GitHub access; set `CARGO_NET_GIT_FETCH_WITH_CLI=true` to use local Git/SSH credentials without putting them in manifests. Cargo may fetch optional dependency metadata even when the feature is off.

## Features

| Feature | Purpose | Notes |
|---|---|---|
| `default` | empty | the core Rust traits need no extra feature |
| `roplat` | enables `robot_behavior::roplat` and the roplat dependency | off by default |
| `ffi` | enables the FFI module gate | base gate |
| `to_py` | `ffi` + `pyo3` | Python binding support |
| `to_cxx` | `ffi` + `cxx` | C++ binding support |
| `to_c` | `ffi` | C-facing gate |

!!! warning
    FFI macros and examples are still migrating. The main documentation follows the current Rust trait API. Before relying on FFI, run `cargo check --features to_py` or the corresponding target feature for your driver.

## Verification Commands

From the `drives` workspace:

```powershell
cargo check -p robot_behavior --no-default-features
cargo check -p robot_behavior --features roplat
cargo test -p robot_behavior --lib
```

From a standalone `robot_behavior` checkout:

```powershell
cargo check --no-default-features
cargo check --features roplat
cargo test --lib
```
