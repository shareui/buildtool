# Building Rust and Cargo (RustBuilder)

The module provides the `RustBuilder` class and the `rust()` factory function to build Rust source files (via `rustc`) or Cargo projects (via `Cargo.toml`) into a static archive (`.a` or `.lib`).

---

## 1. Importing

```yukino
load buildtool: rust, RustBuilder
```

---

## 2. Quick Start

### Option A: Single .rs file via rustc

```yukino
load buildtool: rust

fun main() {
    rust()
        .file("native_rs/myrust.rs")
        .compile("myrust")
}
```

The file `myrust.rs` should export functions using C ABI:

```rust
#[no_mangle]
pub extern "C" fn rust_add(a: i32, b: i32) -> i32 {
    a + b
}
```

### Option B: Cargo project via Cargo.toml

If you have a Cargo project with dependencies:

```toml
# native_cargo/Cargo.toml
[package]
name = "fastmath"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["staticlib"]

[dependencies]
# crate dependencies
```

In your `build.ykn`:

```yukino
load buildtool: rust

fun main() {
    rust()
        .cargo("native_cargo/Cargo.toml")
        .release(true)
        .compile("fastmath")
}
```

The builder runs `cargo build --release --manifest-path native_cargo/Cargo.toml`, copies the built archive (`libfastmath.a` or `fastmath.lib`) into the build output directory, and emits `yuki:link-archive=...`.

---

## 3. RustBuilder Method Reference

### 3.1 file(path: string) -> RustBuilder

Sets the path to a single `.rs` file for compilation with `rustc`.

```yukino
public fun file(path: string) -> RustBuilder
```

### 3.2 cargo(toml_path: string) -> RustBuilder

Sets the path to `Cargo.toml` for building with `cargo`.

```yukino
public fun cargo(toml_path: string) -> RustBuilder
```

### 3.3 release(rel: bool) -> RustBuilder

Switches between `release` (`true`) and `debug` (`false`) modes.

```yukino
public fun release(rel: bool) -> RustBuilder
```

Defaults to `true`.

### 3.4 opt(level: string) -> RustBuilder

Sets the optimization level (`-Copt-level=...` for `rustc`).

```yukino
public fun opt(level: string) -> RustBuilder
```

Defaults to `"3"`.

### 3.5 target(triple: string) -> RustBuilder

Sets the cross-compilation target triple (`--target`).

```yukino
public fun target(triple: string) -> RustBuilder
```

Defaults to the value of `YUKINO_TARGET_TRIPLE` provided by the compiler.

All 7 Yukino targets are supported:
- `x86_64-unknown-linux-gnu`
- `x86_64-unknown-linux-musl`
- `aarch64-unknown-linux-gnu`
- `aarch64-unknown-linux-musl`
- `x86_64-pc-windows-gnu`
- `x86_64-pc-windows-msvc`
- `aarch64-linux-android`

### 3.6 flag(flg: string) -> RustBuilder

Adds an extra flag for `rustc` or `cargo`.

```yukino
public fun flag(flg: string) -> RustBuilder
```

### 3.7 flags(flgs: string[]) -> RustBuilder

Adds multiple flags.

```yukino
public fun flags(flgs: string[]) -> RustBuilder
```

### 3.8 out_dir(dir: string) -> RustBuilder

Sets the output directory for the archive. Defaults to `YUKINO_OUT_DIR` or `yuki-work/local/buildtool`.

```yukino
public fun out_dir(dir: string) -> RustBuilder
```

### 3.9 emit_link_directives(emit: bool) -> RustBuilder

Enables or disables automatic emission of `yuki:link-archive=...`.

```yukino
public fun emit_link_directives(emit: bool) -> RustBuilder
```

Defaults to `true`.

### 3.10 compile(archive_name: string) -> string

Runs compilation and returns the absolute path to the generated archive (`lib<archive_name>.a` or `<archive_name>.lib`).

```yukino
public fun compile(archive_name: string) -> string
```

---

[Back: Building C/C++ (1-cc.md)](1-cc.md) | [Back: Overview (0-overview.md)](0-overview.md)
