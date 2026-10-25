# buildtool Library

The `buildtool` library is a native build helper for the Yukino programming language and `build.ykn` build scripts. It provides a high-level, ergonomic API to compile C, C++, and Rust source files into static archives (`.a` or `.lib`) and pass linker directives to Yukino.

The library fully supports Linux and Windows host build machines, and cross-compilation for all 7 targets supported by the `yukic` compiler:
1. `x86_64-unknown-linux-gnu`
2. `x86_64-unknown-linux-musl`
3. `aarch64-unknown-linux-gnu`
4. `aarch64-unknown-linux-musl`
5. `x86_64-pc-windows-gnu`
6. `x86_64-pc-windows-msvc`
7. `aarch64-linux-android`

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
- **Toolchain detection:**
  - `find_c_compiler`, `find_c_compiler_for_target`
  - `find_cpp_compiler`, `find_cpp_compiler_for_target`
  - `find_ar`, `find_ar_for_target`
  - `find_rustc`, `find_cargo`
  - `tool_exists`
- **Platform and tool predicates:** `is_msvc_target`, `is_windows_target`, `is_msvc_compiler`, `is_msvc_archiver`, `is_clang_compiler`.
- **Error structures:** `ToolchainError`, `CompilationError`, `ConfigError`.

---

## 2. Architecture and How It Works

The `build.ykn` script is executed by `yukino` before building the main project. `buildtool`:
1. Automatically reads the target platform from the `YUKINO_TARGET_TRIPLE` environment variable (passed by `yukino`), or uses the platform specified via `.target(triple)`.
2. Locates an appropriate compiler and archiver for the target platform and host machine:
   - **Clang / LLVM:** `clang`, `clang++`, `llvm-ar`, `llvm-lib` (universal toolchain for all 7 targets).
   - **GNU / MinGW:** `gcc`, `g++`, `ar`, `x86_64-w64-mingw32-gcc`, `aarch64-linux-gnu-gcc`, `musl-gcc`, `aarch64-linux-android-clang`.
   - **MSVC (Windows):** `cl.exe`, `lib.exe`, `clang-cl`.
   - **Rust:** `rustc`, `cargo`.
3. If a tool is missing, raises a clear `ToolchainError` with installation guidance.
4. Compiles sources following target platform conventions:
   - For ELF platforms (Linux/Android), adds `-fPIC`.
   - For MSVC, uses `/c`, `/Fo`, `/O2` and the `lib /OUT:` archiver.
   - For Clang cross-compilation, adds `--target=<triple>`.
5. Archives the resulting object files into a static library:
   - `lib<name>.a` for Linux, Android, and Windows GNU.
   - `<name>.lib` for Windows MSVC.
6. Emits standard build directives to `stdout`:
   - `yuki:link-archive=...` — instructs Yukino to link the archive.
   - `yuki:link-system=stdc++` (or `libcpmt` for MSVC) — automatically links C++ runtime when compiling C++ sources.
7. The Yukino driver captures these directives and links the archives into your final binary.

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
- `1001`: C compiler not found for target.
- `1002`: C++ compiler not found for target.
- `1003`: Archiver tool (`ar`, `llvm-ar`, or `lib`) not found.
- `1004`: `rustc` compiler not found.
- `1005`: `cargo` package manager not found.
- `2001`: No source files provided for C/C++.
- `2002`: C/C++ compilation command failed.
- `2003`: Archiver command failed.
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
