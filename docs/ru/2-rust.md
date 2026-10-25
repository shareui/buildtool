# Сборка Rust кода (RustBuilder)

Модуль предоставляет класс `RustBuilder` и фабричную функцию `rust()` для сборки исходных файлов Rust (через `rustc`) или полноценных проектов Cargo (через `Cargo.toml`) в статический архив (`.a` или `.lib`).

---

## 1. Подключение

```yukino
load buildtool: rust, RustBuilder
```

---

## 2. Быстрый старт

### Вариант А: Одиночный .rs файл через rustc

```yukino
load buildtool: rust

fun main() {
    rust()
        .file("native_rs/myrust.rs")
        .compile("myrust")
}
```

Файл `myrust.rs` должен содержать функции с C ABI:

```rust
#[no_mangle]
pub extern "C" fn rust_add(a: i32, b: i32) -> i32 {
    a + b
}
```

### Вариант Б: Проект Cargo через Cargo.toml

Если у вас есть проект Cargo с внешними зависимостями:

```toml
# native_cargo/Cargo.toml
[package]
name = "fastmath"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["staticlib"]

[dependencies]
# зависимости crate
```

В вашем `build.ykn`:

```yukino
load buildtool: rust

fun main() {
    rust()
        .cargo("native_cargo/Cargo.toml")
        .release(true)
        .compile("fastmath")
}
```

Билдер запустит `cargo build --release --manifest-path native_cargo/Cargo.toml`, скопирует собранный архив (`libfastmath.a` или `fastmath.lib`) в директорию `yuki-work/local/build/...` и выведет директиву `yuki:link-archive=...`.

---

## 3. Справочник методов класса RustBuilder

### 3.1 file(path: string) -> RustBuilder

Задает путь к одиночному `.rs` файлу для компиляции через `rustc`.

```yukino
public fun file(path: string) -> RustBuilder
```

### 3.2 cargo(toml_path: string) -> RustBuilder

Задает путь к манифесту `Cargo.toml` для сборки через `cargo`.

```yukino
public fun cargo(toml_path: string) -> RustBuilder
```

### 3.3 release(rel: bool) -> RustBuilder

Переключает режим сборки между `release` (`true`) и `debug` (`false`).

```yukino
public fun release(rel: bool) -> RustBuilder
```

По умолчанию включен режим `true`.

### 3.4 opt(level: string) -> RustBuilder

Задает уровень оптимизации (для `rustc` передается флаг `-Copt-level=...`).

```yukino
public fun opt(level: string) -> RustBuilder
```

По умолчанию `"3"`.

### 3.5 target(triple: string) -> RustBuilder

Задает целевую платформу для кросс-компиляции (флаг `--target`).

```yukino
public fun target(triple: string) -> RustBuilder
```

По умолчанию платформа автоматически считывается из переменной окружения `YUKINO_TARGET_TRIPLE`, которую компилятор Yukino передает в `build.ykn`.

Поддерживаются все 7 целевых платформ:
- `x86_64-unknown-linux-gnu`
- `x86_64-unknown-linux-musl`
- `aarch64-unknown-linux-gnu`
- `aarch64-unknown-linux-musl`
- `x86_64-pc-windows-gnu`
- `x86_64-pc-windows-msvc`
- `aarch64-linux-android`

### 3.6 flag(flg: string) -> RustBuilder

Добавляет дополнительный флаг для `rustc` или `cargo`.

```yukino
public fun flag(flg: string) -> RustBuilder
```

### 3.7 flags(flgs: string[]) -> RustBuilder

Добавляет список дополнительных флагов.

```yukino
public fun flags(flgs: string[]) -> RustBuilder
```

### 3.8 out_dir(dir: string) -> RustBuilder

Задает каталог вывода статического архива. По умолчанию используется каталог сборки Yukino (`YUKINO_OUT_DIR` или `yuki-work/local/buildtool`).

```yukino
public fun out_dir(dir: string) -> RustBuilder
```

### 3.9 emit_link_directives(emit: bool) -> RustBuilder

Включает или выключает автоматический вывод директивы `yuki:link-archive=...` в стандартный вывод.

```yukino
public fun emit_link_directives(emit: bool) -> RustBuilder
```

По умолчанию включено (`true`).

### 3.10 compile(archive_name: string) -> string

Запускает процесс компиляции и возвращает абсолютный путь к собранному архиву (`lib<archive_name>.a` или `<archive_name>.lib` для MSVC).

```yukino
public fun compile(archive_name: string) -> string
```

---

[Назад: Сборка C/C++ (1-cc.md)](1-cc.md) | [В начало: Обзор](0-overview.md)
