# buildtool

A native build helper library for the Yukino programming language.

`buildtool` is designed for pre-build scripts (`build.ykn`) to compile native C, C++, and Rust code into static archives (`.a`), seamlessly connecting them to your Yukino project.

---

## Features

- **C and C++ Compilation**: Compile `.c`, `.cpp`, `.cc`, `.cxx` files using system toolchains (`clang`, `gcc`, `clang++`, `g++`).
- **Rust Integration**: Compile standalone `.rs` source files directly via `rustc`, or build entire Rust crates via `cargo`.
- **Static Archives**: Produces static archives (`.a`) exclusively; does not emit shared libraries (`.so`).
- **Automatic Linker Directives**: Automatically emits `yuki:link-archive=<path>` and system dependencies (such as `stdc++`) to standard output for the Yukino compiler.
- **Clean Workspace Integration**: Writes artifacts into `$YUKINO_OUT_DIR/native/`, fully compatible with `yukino clean` and `--rebuild`.
- **Structured Error Handling**: Throws typed panics (`ToolchainError`, `CompilationError`, `ConfigError`) with actionable diagnostics when tools are missing or builds fail.

---

## Installation

Add `buildtool` to your project manifest under the `[libs.build]` section in `yuki.toml`:

```toml
[libs.build]
buildtool = { git = "https://github.com/shareui/buildtool" }
```

> Dependencies declared in `[libs.build]` are compiled for the host platform and made available inside `build.ykn`.

---

## Quick Example

Create a `build.ykn` file in the root of your project:

```yukino
load buildtool.cc: cc
load buildtool.rust: rust

public fun main() {
    // Compile C/C++ files into libnative.a
    cc()
        .file("native/add.c")
        .file("native/multiply.cpp")
        .include("native/include")
        .opt("2")
        .compile("native")

    // Compile a Rust crate into librustlogic.a
    rust()
        .cargo("rust_crate/Cargo.toml")
        .release(true)
        .compile("rustlogic")
}
```

Now, functions declared with `externc` in your Yukino code will be resolved automatically at link time.

---

## Documentation

Full documentation is available in English and Russian:

- **English Documentation**:
  - [Overview and Quick Start](docs/en/0-overview.md)
  - [C and C++ Builder Guide (`cc`)](docs/en/1-cc.md)
  - [Rust Builder Guide (`rust`)](docs/en/2-rust.md)

- **Russian Documentation**:
  - [Обзор и быстрый старт](docs/ru/0-overview.md)
  - [Руководство по C/C++ (`cc`)](docs/ru/1-cc.md)
  - [Руководство по Rust (`rust`)](docs/ru/2-rust.md)

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

Copyright (c) 2026 shareui.
