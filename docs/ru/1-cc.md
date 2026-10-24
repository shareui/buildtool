# Сборка C и C++ кода (CcBuilder)

Модуль предоставляет класс `CcBuilder` и фабричную функцию `cc()` для гибкой компиляции исходников на C и C++ в статический архив `.a`.

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
1. Файл заголовка `native/mymath.h` автоматически регистрирует родительскую директорию `native/` в путях поиска (`-I native`).
2. Исходный файл `native/mymath.c` компилируется в объектный файл.
3. Утилита `ar` упаковывает объектный файл в `libmymath.a`.
4. Билдер автоматически выводит директиву `yuki:link-archive=...`, и итоговый бинарник Yukino успешно линкуется с `mymath`.

---

## 3. Справочник методов класса CcBuilder

### 3.1 file(path: string) -> CcBuilder

Добавляет один файл в сборку.

```yukino
public fun file(path: string) -> CcBuilder
```

- Если передается файл заголовков (`.h` или `.hpp`), он не компилируется отдельно, а его родительская папка автоматически добавляется в список путей поиска инклудов (`-I`).
- Если передается файл с расширением `.cpp`, `.cc`, `.cxx` или `.C`, билдер автоматически активирует режим компилятора C++ (`is_cpp = true`).

### 3.2 files(paths: string[]) -> CcBuilder

Добавляет список файлов в сборку.

```yukino
public fun files(paths: string[]) -> CcBuilder
```

### 3.3 include(dir: string) -> CcBuilder

Добавляет директорию в пути поиска заголовков (флаг `-I`).

```yukino
public fun include(dir: string) -> CcBuilder
```

### 3.4 includes(dirs: string[]) -> CcBuilder

Добавляет список директорий для поиска заголовков.

```yukino
public fun includes(dirs: string[]) -> CcBuilder
```

### 3.5 def(name: string, val: string?) -> CcBuilder

Определяет препроцессорный макрос (флаг `-D`).

```yukino
public fun def(name: string, val: string?) -> CcBuilder
```

- Если `val` равен `null`, передается флаг `-D<name>`.
- Если `val` задан, передается флаг `-D<name>=<val>`.

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

Задает стандарт языка (флаг `-std=...`).

```yukino
public fun std(version: string) -> CcBuilder
```

Например: `.std("c11")` или `.std("c++17")`.

### 3.9 opt(level: string) -> CcBuilder

Задает уровень оптимизации компилятора C/C++ (флаг `-O...`).

```yukino
public fun opt(level: string) -> CcBuilder
```

По умолчанию используется уровень `"2"`. Например: `.opt("3")` или `.opt("s")`.

### 3.10 cpp(enabled: bool) -> CcBuilder

Явно принудительно включает или выключает режим C++.

```yukino
public fun cpp(enabled: bool) -> CcBuilder
```

### 3.11 out_dir(dir: string) -> CcBuilder

Задает директорию для сохранения скомпилированных объектных файлов и архива.

```yukino
public fun out_dir(dir: string) -> CcBuilder
```

Если не задана, автоматически используется переменная окружения `YUKINO_OUT_DIR`.

### 3.12 emit_link_directives(emit: bool) -> CcBuilder

Включает или отключает автоматический вывод сборочных директив в стандартный вывод `stdout`.

```yukino
public fun emit_link_directives(emit: bool) -> CcBuilder
```

По умолчанию включено (`true`).

### 3.13 compile(archive_name: string) -> string

Запускает процесс компиляции и создания архива.

```yukino
public fun compile(archive_name: string) -> string
```

Возвращает абсолютный путь к созданному файлу `lib<archive_name>.a`.  
При успехе автоматически печатает директиву `yuki:link-archive=<путь>` и (в случае C++) `yuki:link-system=stdc++`.

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
        .flag("-Wall")
        .compile("native_wrapper")
}
```

---

[Назад: Обзор](0-overview.md) | [Вперед: Сборка Rust (2-rust.md)](2-rust.md)
