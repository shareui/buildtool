# Building C and C++ (CcBuilder)

The module provides the `CcBuilder` class and the `cc()` factory function for flexible compilation of C and C++ sources into a static archive (`.a` or `.lib`).

---

## 1. Importing

```yukino
load buildtool: cc, CcBuilder
```

---

## 2. Quick Start

A simple `build.ykn` at the root of your project:

```yukino
load buildtool: cc

fun main() {
    cc()
        .file("native/mymath.h")
        .file("native/mymath.c")
        .compile("mymath")
}
```

When running `yukino build` or `yukino run`:
1. Header file `native/mymath.h` automatically adds its parent directory `native/` to include search paths (`-I native` or `/I native`).
2. Source file `native/mymath.c` is compiled into an object file (`.o` or `.obj`).
3. The archiver (`ar`, `llvm-ar`, or `lib.exe`) packs the object file into a static archive (`libmymath.a` or `mymath.lib`).
4. The builder emits `yuki:link-archive=...`, and the Yukino binary links with the native library.

---

## 3. CcBuilder Method Reference

### 3.1 file(path: string) -> CcBuilder

Adds a single file to the build.

```yukino
public fun file(path: string) -> CcBuilder
```

- If a header file (`.h` or `.hpp`) is passed, its parent folder is added to include paths (`-I` or `/I`).
- If a C++ source file (`.cpp`, `.cc`, `.cxx`, `.C`) is passed, C++ mode is activated automatically (`is_cpp = true`).

### 3.2 files(paths: string[]) -> CcBuilder

Adds multiple files to the build.

```yukino
public fun files(paths: string[]) -> CcBuilder
```

### 3.3 include(dir: string) -> CcBuilder

Adds an include directory (flag `-I` or `/I`).

```yukino
public fun include(dir: string) -> CcBuilder
```

### 3.4 includes(dirs: string[]) -> CcBuilder

Adds multiple include directories.

```yukino
public fun includes(dirs: string[]) -> CcBuilder
```

### 3.5 def(name: string, val: string?) -> CcBuilder

Defines a preprocessor macro (flag `-D` or `/D`).

```yukino
public fun def(name: string, val: string?) -> CcBuilder
```

- If `val` is `null`, emits `-D<name>` or `/D<name>`.
- If `val` is provided, emits `-D<name>=<val>` or `/D<name>=<val>`.

### 3.6 flag(flg: string) -> CcBuilder

Adds an arbitrary compiler flag.

```yukino
public fun flag(flg: string) -> CcBuilder
```

### 3.7 flags(flgs: string[]) -> CcBuilder

Adds multiple arbitrary compiler flags.

```yukino
public fun flags(flgs: string[]) -> CcBuilder
```

### 3.8 std(version: string) -> CcBuilder

Sets the language standard (`-std=...` for GCC/Clang, `/std:...` for MSVC).

```yukino
public fun std(version: string) -> CcBuilder
```

Examples: `.std("c11")` or `.std("c++17")`.

### 3.9 opt(level: string) -> CcBuilder

Sets the optimization level.

```yukino
public fun opt(level: string) -> CcBuilder
```

Defaults to `"2"` (`-O2` or `/O2`). Other levels like `"0"`, `"1"`, `"3"`, `"s"` are supported.

### 3.10 target(triple: string) -> CcBuilder

Sets the cross-compilation target triple.

```yukino
public fun target(triple: string) -> CcBuilder
```

By default, the target is automatically detected from the `YUKINO_TARGET_TRIPLE` environment variable passed by the Yukino driver.

All 7 Yukino targets are supported:
- `x86_64-unknown-linux-gnu`
- `x86_64-unknown-linux-musl`
- `aarch64-unknown-linux-gnu`
- `aarch64-unknown-linux-musl`
- `x86_64-pc-windows-gnu`
- `x86_64-pc-windows-msvc`
- `aarch64-linux-android`

### 3.11 compiler(path: string) -> CcBuilder

Explicitly sets the compiler binary path.

```yukino
public fun compiler(path: string) -> CcBuilder
```

### 3.12 archiver(path: string) -> CcBuilder

Explicitly sets the static archiver binary path (`ar`, `llvm-ar`, `lib.exe`).

```yukino
public fun archiver(path: string) -> CcBuilder
```

### 3.13 cpp(enabled: bool) -> CcBuilder

Explicitly enables or disables C++ compilation mode.

```yukino
public fun cpp(enabled: bool) -> CcBuilder
```

### 3.14 out_dir(dir: string) -> CcBuilder

Sets the output directory for object files and archives. Defaults to `YUKINO_OUT_DIR` or `yuki-work/local/buildtool`.

```yukino
public fun out_dir(dir: string) -> CcBuilder
```

### 3.15 emit_link_directives(emit: bool) -> CcBuilder

Enables or disables automatic emission of `yuki:link-*` directives to stdout.

```yukino
public fun emit_link_directives(emit: bool) -> CcBuilder
```

Defaults to `true`.

### 3.16 compile(archive_name: string) -> string

Compiles sources and packs them into a static archive.

```yukino
public fun compile(archive_name: string) -> string
```

Returns the absolute path to the generated archive (`lib<archive_name>.a` or `<archive_name>.lib`).  
Emits `yuki:link-archive=<path>` and (for C++) `yuki:link-system=stdc++` (or `libcpmt` for MSVC).

---

## 4. C++ Example

```yukino
load buildtool: cc

fun main() {
    cc()
        .file("cpp_src/wrapper.hpp")
        .file("cpp_src/wrapper.cpp")
        .std("c++20")
        .opt("3")
        .def("ENABLE_FAST_MATH", "1")
        .compile("native_wrapper")
}
```

---

[Back: Overview](0-overview.md) | [Next: Building Rust (2-rust.md)](2-rust.md)
