# Building C and C++ (CcBuilder)

The `CcBuilder` class and its `cc()` factory function provide a flexible interface for compiling C and C++ sources into static `.a` archives.

---

## 1. Import

```yukino
load buildtool: cc, CcBuilder
```

---

## 2. Quick Start

Create a `build.ykn` file in your project root:

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
1. Passing `native/mymath.h` registers `native/` in the include search paths (`-I native`).
2. `native/mymath.c` is compiled into an object file.
3. The `ar` utility packages the object file into `libmymath.a`.
4. The builder outputs `yuki:link-archive=...`, causing Yukino to link `mymath` into the application binary.

---

## 3. CcBuilder Method Reference

### 3.1 file(path: string) -> CcBuilder

Adds a single file to the compilation.

```yukino
public fun file(path: string) -> CcBuilder
```

- Header files (`.h` or `.hpp`) are not compiled directly; their parent directory is automatically added to the include search paths (`-I`).
- C++ files (`.cpp`, `.cc`, `.cxx`, `.C`) automatically enable C++ compilation mode (`is_cpp = true`).

### 3.2 files(paths: string[]) -> CcBuilder

Adds multiple files to the compilation.

```yukino
public fun files(paths: string[]) -> CcBuilder
```

### 3.3 include(dir: string) -> CcBuilder

Adds a directory to header search paths (`-I`).

```yukino
public fun include(dir: string) -> CcBuilder
```

### 3.4 includes(dirs: string[]) -> CcBuilder

Adds multiple directories to header search paths.

```yukino
public fun includes(dirs: string[]) -> CcBuilder
```

### 3.5 def(name: string, val: string?) -> CcBuilder

Defines a preprocessor macro (`-D`).

```yukino
public fun def(name: string, val: string?) -> CcBuilder
```

- If `val` is `null`, `-D<name>` is passed.
- If `val` is provided, `-D<name>=<val>` is passed.

### 3.6 flag(flg: string) -> CcBuilder

Appends a custom compiler flag.

```yukino
public fun flag(flg: string) -> CcBuilder
```

### 3.7 flags(flgs: string[]) -> CcBuilder

Appends multiple compiler flags.

```yukino
public fun flags(flgs: string[]) -> CcBuilder
```

### 3.8 std(version: string) -> CcBuilder

Specifies the language standard (`-std=...`), e.g., `.std("c11")` or `.std("c++20")`.

```yukino
public fun std(version: string) -> CcBuilder
```

### 3.9 opt(level: string) -> CcBuilder

Sets the optimization level (`-O...`). Default is `"2"`.

```yukino
public fun opt(level: string) -> CcBuilder
```

### 3.10 cpp(enabled: bool) -> CcBuilder

Explicitly enables or disables C++ compilation mode.

```yukino
public fun cpp(enabled: bool) -> CcBuilder
```

### 3.11 out_dir(dir: string) -> CcBuilder

Specifies the output directory for artifacts. Defaults to `YUKINO_OUT_DIR`.

```yukino
public fun out_dir(dir: string) -> CcBuilder
```

### 3.12 emit_link_directives(emit: bool) -> CcBuilder

Controls whether `yuki:link-archive=` and `yuki:link-system=` directives are printed to stdout. Default is `true`.

```yukino
public fun emit_link_directives(emit: bool) -> CcBuilder
```

### 3.13 compile(archive_name: string) -> string

Compiles all source files, creates `lib<archive_name>.a`, emits link directives, and returns the absolute archive path.

```yukino
public fun compile(archive_name: string) -> string
```

---

## 4. Full C++ Example

```yukino
load buildtool: cc

fun main() {
    cc()
        .file("cpp_src/wrapper.hpp")
        .file("cpp_src/wrapper.cpp")
        .std("c++20")
        .opt("3")
        .def("ENABLE_FAST_MATH", "1")
        .flag("-Wall")
        .compile("native_wrapper")
}
```

---

[Back: Overview](0-overview.md) | [Next: Rust Builder (2-rust.md)](2-rust.md)
