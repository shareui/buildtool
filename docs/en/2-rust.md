# Building Rust Code (RustBuilder)

The `RustBuilder` class and its `rust()` factory function compile single Rust source files (via `rustc`) or Cargo projects (via `Cargo.toml`) into static archives (`.a`).

---

## 1. Import

```yukino
load buildtool: rust, RustBuilder
```

---

## 2. Quick Start

### Approach A: Single .rs file via rustc

```yukino
load buildtool: rust

fun main() {
    rust()
        .file("native_rs/myrust.rs")
        .compile("myrust")
}
```

The Rust file should export symbols using the C ABI:

```rust
#[no_mangle]
pub extern "C" fn rust_add(a: i32, b: i32) -> i32 {
    a + b
}
```

### Approach B: Cargo project via Cargo.toml

For Cargo projects with dependencies:

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

The builder runs `cargo build --release --manifest-path native_cargo/Cargo.toml`, copies `libfastmath.a` to the output directory, and emits `yuki:link-archive=...`.

---

## 3. RustBuilder Method Reference

### 3.1 file(path: string) -> RustBuilder

Sets the path to a single `.rs` file compiled via `rustc`.

```yukino
public fun file(path: string) -> RustBuilder
```

### 3.2 cargo(toml_path: string) -> RustBuilder

Sets the path to a `Cargo.toml` manifest compiled via `cargo`.

```yukino
public fun cargo(toml_path: string) -> RustBuilder
```

### 3.3 release(rel: bool) -> RustBuilder

Toggles between `release` (`true`) and `debug` (`false`) build modes. Defaults to `true`.

```yukino
public fun release(rel: bool) -> RustBuilder
```

### 3.4 opt(level: string) -> RustBuilder

Sets the optimization level (`-Copt-level=...` for `rustc`). Defaults to `"3"`.

```yukino
public fun opt(level: string) -> RustBuilder
```

### 3.5 target(triple: string) -> RustBuilder

Sets the target triple for cross-compilation (`--target`).

```yukino
public fun target(triple: string) -> RustBuilder
```

### 3.6 flag(flg: string) -> RustBuilder

Adds an additional flag to `rustc` or `cargo`.

```yukino
public fun flag(flg: string) -> RustBuilder
```

### 3.7 flags(flgs: string[]) -> RustBuilder

Adds multiple additional flags.

```yukino
public fun flags(flgs: string[]) -> RustBuilder
```

### 3.8 out_dir(dir: string) -> RustBuilder

Sets the destination directory for the archive. Defaults to `YUKINO_OUT_DIR`.

```yukino
public fun out_dir(dir: string) -> RustBuilder
```

### 3.9 emit_link_directives(emit: bool) -> RustBuilder

Controls whether `yuki:link-archive=` is printed to stdout. Defaults to `true`.

```yukino
public fun emit_link_directives(emit: bool) -> RustBuilder
```

### 3.10 compile(archive_name: string) -> string

Executes compilation and returns the absolute path to `lib<archive_name>.a`.

```yukino
public fun compile(archive_name: string) -> string
```

---

[Back: Building C/C++ (1-cc.md)](1-cc.md) | [Back to Overview](0-overview.md)
