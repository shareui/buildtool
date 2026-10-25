# Сборка C и C++ кода (CcBuilder)

Модуль предоставляет класс `CcBuilder` и фабричную функцию `cc()` для гибкой компиляции исходников на C и C++ в статический архив (`.a` или `.lib`).

---

## 1. Подключение

```yukino
load buildtool: cc, CcBuilder
```

---

## 2. Быстрый старт

Пример простого `build.ykn` в корне вашего проекта:

```yukino
load buildtool: cc

fun main() {
    cc()
        .file("native/mymath.h")
        .file("native/mymath.c")
        .compile("mymath")
}
```

При запуске `yukino build` или `yukino run`:
1. Файл заголовка `native/mymath.h` автоматически регистрирует родительскую директорию `native/` в путях поиска (`-I native` или `/I native`).
2. Исходный файл `native/mymath.c` компилируется в объектный файл (`.o` или `.obj`).
3. Архиватор (`ar`, `llvm-ar` или `lib.exe`) упаковывает объектный файл в статический архив (`libmymath.a` или `mymath.lib`).
4. Билдер автоматически выводит директиву `yuki:link-archive=...`, и итоговый бинарник Yukino линкуется с библиотекой.

---

## 3. Справочник методов класса CcBuilder

### 3.1 file(path: string) -> CcBuilder

Добавляет один файл в сборку.

```yukino
public fun file(path: string) -> CcBuilder
```

- Если передается файл заголовков (`.h` или `.hpp`), его родительская папка автоматически добавляется в список путей поиска инклудов (`-I` или `/I`).
- Если передается файл с расширением `.cpp`, `.cc`, `.cxx` или `.C`, билдер автоматически активирует режим компилятора C++ (`is_cpp = true`).

### 3.2 files(paths: string[]) -> CcBuilder

Добавляет список файлов в сборку.

```yukino
public fun files(paths: string[]) -> CcBuilder
```

### 3.3 include(dir: string) -> CcBuilder

Добавляет директорию в пути поиска заголовков (флаг `-I` или `/I`).

```yukino
public fun include(dir: string) -> CcBuilder
```

### 3.4 includes(dirs: string[]) -> CcBuilder

Добавляет список директорий для поиска заголовков.

```yukino
public fun includes(dirs: string[]) -> CcBuilder
```

### 3.5 def(name: string, val: string?) -> CcBuilder

Определяет препроцессорный макрос (флаг `-D` или `/D`).

```yukino
public fun def(name: string, val: string?) -> CcBuilder
```

- Если `val` равен `null`, передается флаг `-D<name>` или `/D<name>`.
- Если `val` задан, передается флаг `-D<name>=<val>` или `/D<name>=<val>`.

### 3.6 flag(flg: string) -> CcBuilder

Добавляет произвольный флаг командной строки компилятора.

```yukino
public fun flag(flg: string) -> CcBuilder
```

### 3.7 flags(flgs: string[]) -> CcBuilder

Добавляет список произвольных флагов компилятора.

```yukino
public fun flags(flgs: string[]) -> CcBuilder
```

### 3.8 std(version: string) -> CcBuilder

Задает стандарт языка (флаг `-std=...` для GCC/Clang или `/std:...` для MSVC).

```yukino
public fun std(version: string) -> CcBuilder
```

Например: `.std("c11")` или `.std("c++17")`.

### 3.9 opt(level: string) -> CcBuilder

Задает уровень оптимизации компилятора C/C++.

```yukino
public fun opt(level: string) -> CcBuilder
```

По умолчанию используется уровень `"2"` (`-O2` или `/O2`). Также поддерживаются `"0"`, `"1"`, `"3"`, `"s"`.

### 3.10 target(triple: string) -> CcBuilder

Задает целевую платформу кросс-компиляции.

```yukino
public fun target(triple: string) -> CcBuilder
```

По умолчанию целевая платформа автоматически считывается из переменной окружения `YUKINO_TARGET_TRIPLE`, которую компилятор Yukino передает в `build.ykn`. Метод позволяет явно переопределить платформу при необходимости.

Поддерживаются все 7 платформ компилятора:
- `x86_64-unknown-linux-gnu`
- `x86_64-unknown-linux-musl`
- `aarch64-unknown-linux-gnu`
- `aarch64-unknown-linux-musl`
- `x86_64-pc-windows-gnu`
- `x86_64-pc-windows-msvc`
- `aarch64-linux-android`

### 3.11 compiler(path: string) -> CcBuilder

Явно задает путь к исполняемому файлу компилятора C/C++.

```yukino
public fun compiler(path: string) -> CcBuilder
```

Если не задан, компилятор выбирается автоматически на основе целевой платформы и установленных в системе инструментов.

### 3.12 archiver(path: string) -> CcBuilder

Явно задает путь к утилите создания статических архивов (`ar`, `llvm-ar`, `lib.exe`).

```yukino
public fun archiver(path: string) -> CcBuilder
```

### 3.13 cpp(enabled: bool) -> CcBuilder

Явно принудительно включает или выключает режим C++.

```yukino
public fun cpp(enabled: bool) -> CcBuilder
```

### 3.14 out_dir(dir: string) -> CcBuilder

Задает директорию для сохранения скомпилированных объектных файлов и архива.

```yukino
public fun out_dir(dir: string) -> CcBuilder
```

Если не задана, автоматически используется переменная окружения `YUKINO_OUT_DIR`.

### 3.15 emit_link_directives(emit: bool) -> CcBuilder

Включает или отключает автоматический вывод сборочных директив в стандартный вывод `stdout`.

```yukino
public fun emit_link_directives(emit: bool) -> CcBuilder
```

По умолчанию включено (`true`).

### 3.16 compile(archive_name: string) -> string

Запускает процесс компиляции и создания архива.

```yukino
public fun compile(archive_name: string) -> string
```

Возвращает абсолютный путь к созданному файлу архива (`lib<archive_name>.a` или `<archive_name>.lib`).  
При успехе автоматически печатает директиву `yuki:link-archive=<путь>` и (в случае C++) директиву `yuki:link-system=stdc++` (или `libcpmt` для MSVC).

---

## 4. Комплексный пример для C++

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

[Назад: Обзор](0-overview.md) | [Вперед: Сборка Rust (2-rust.md)](2-rust.md)
