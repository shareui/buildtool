# buildtool Library

The `buildtool` library is a native build helper for the Yukino programming language and `build.ykn` build scripts. It provides a high-level, ergonomic API to compile C, C++, and Rust source files into static archives (`.a`) and pass linker directives to Yukino.

---

## 1. Installation: load buildtool

The library is designed for use in the pre-build script `build.ykn`. Add it to the `[libs.build]` section of your `yuki.toml`:

```toml
[libs.build]
buildtool = { git = "https://github.com/shareui/buildtool", tag = "v0.1.0" }
```

Or using a local path:

```toml
[libs.build]
buildtool = { path = "../libs/buildtool" }
```

### Importing in build.ykn

```yukino
load buildtool: cc, rust
```

Available symbols:
- **Builder constructors:** `cc()`, `rust()`.
- **Linker directive helpers:** `emit_link_archive`, `emit_link_system`, `emit_warning`.
- **Toolchain detection:** `find_c_compiler`, `find_cpp_compiler`, `find_ar`, `find_rustc`, `find_cargo`.
- **Error structures:** `ToolchainError`, `CompilationError`, `ConfigError`.

---

## 2. Architecture and How It Works

The `build.ykn` script is executed by `yukino` before building the main project. `buildtool`:
1. Detects available compilers in `PATH` (`clang` and `gcc` for C/C++; `rustc` and `cargo` for Rust).
2. If a required compiler is missing, it raises a clear error advising which tool to install.
3. Compiles sources with `-fPIC` and optimization flags.
4. Archives the resulting object files into a static library (`lib<name>.a`) using `ar`.
5. Emits standard build directives to `stdout`:
   - `yuki:link-archive=...` — instructs Yukino to link the `.a` archive (or `.o` object file).
   - `yuki:link-system=stdc++` — automatically linked when compiling C++ sources.
6. The Yukino driver captures these directives and links the archives into your final binary.

---

## 3. Error Structures

When a toolchain or compilation problem occurs, `buildtool` panics with strongly-typed error structures:

```yukino
public struct ToolchainError {
    tool: string
    message: string
    code: int32
}

public struct CompilationError {
    message: string
    command: string
    output: string
    code: int32
}

public struct ConfigError {
    message: string
    code: int32
}
```

Error codes:
- `1001`: C compiler not found (`clang` or `gcc`).
- `1002`: C++ compiler not found (`clang++` or `g++`).
- `1003`: Archiver `ar` not found.
- `1004`: `rustc` compiler not found.
- `1005`: `cargo` package manager not found.
- `2001`: No source files provided for C/C++.
- `2002`: C/C++ compilation command failed.
- `2003`: Archiver `ar` command failed.
- `3001`: `cargo build` command failed.
- `3002`: Built Cargo artifact not found in target directory.
- `3003`: `rustc` compilation command failed.
- `3004`: No Rust source file or Cargo.toml specified.

---

## 4. Documentation Index

- [1. Building C and C++ (CcBuilder)](1-cc.md)
- [2. Building Rust and Cargo (RustBuilder)](2-rust.md)

---

[Next: Building C and C++ (1-cc.md)](1-cc.md)
