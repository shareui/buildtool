# Библиотека buildtool

Библиотека `buildtool` — инструмент сборки нативного кода для языка Yukino и сборочных скриптов `build.ykn`. Она предоставляет высокоуровневый и эргономичный программный интерфейс для автоматической компиляции исходных файлов на C, C++ и Rust, упаковки их в статические библиотеки (`.a` или `.lib`) и автоматической передачи директив линковщику Yukino.

Библиотека полностью поддерживает работу на хост-системах Linux и Windows, а также кросс-компиляцию для всех 7 целевых платформ компилятора `yukic`:
1. `x86_64-unknown-linux-gnu`
2. `x86_64-unknown-linux-musl`
3. `aarch64-unknown-linux-gnu`
4. `aarch64-unknown-linux-musl`
5. `x86_64-pc-windows-gnu`
6. `x86_64-pc-windows-msvc`
7. `aarch64-linux-android`

---

## 1. Подключение: load buildtool

Библиотека используется в предсборочном скрипте `build.ykn`. Чтобы подключить ее, добавьте запись в секцию `[libs.build]` манифеста `yuki.toml`:

```toml
[libs.build]
buildtool = { git = "https://github.com/shareui/buildtool", tag = "v0.1.1" }
```

Или по локальному пути:

```toml
[libs.build]
buildtool = { path = "../libs/buildtool" }
```

### Импорт в коде build.ykn

```yukino
load buildtool: cc, rust
```

Доступные сущности:
- **Функции создания билдеров:** `cc()`, `rust()`.
- **Вспомогательные функции вывода директив:** `emit_link_archive`, `emit_link_system`, `emit_warning`.
- **Поиск и проверки тулчейнов:**
  - `find_c_compiler`, `find_c_compiler_for_target`
  - `find_cpp_compiler`, `find_cpp_compiler_for_target`
  - `find_ar`, `find_ar_for_target`
  - `find_rustc`, `find_cargo`
  - `tool_exists`
- **Предикаты платформ и компиляторов:** `is_msvc_target`, `is_windows_target`, `is_msvc_compiler`, `is_msvc_archiver`, `is_clang_compiler`.
- **Структуры ошибок:** `ToolchainError`, `CompilationError`, `ConfigError`.

---

## 2. Архитектура и принцип работы

Скрипт `build.ykn` запускается компилятором `yukino` перед началом сборки приложения. Библиотека `buildtool`:
1. Автоматически считывает целевую платформу из переменной окружения `YUKINO_TARGET_TRIPLE` (передается компилятором при сборке), либо использует платформу, заданную вызовом `.target(triple)`.
2. Находит подходящий компилятор и архиватор для целевой платформы и хост-системы:
   - **Clang / LLVM:** `clang`, `clang++`, `llvm-ar`, `llvm-lib` (универсальный тулчейн для всех 7 платформ).
   - **GNU / MinGW:** `gcc`, `g++`, `ar`, `x86_64-w64-mingw32-gcc`, `aarch64-linux-gnu-gcc`, `musl-gcc`, `aarch64-linux-android-clang`.
   - **MSVC (Windows):** `cl.exe`, `lib.exe`, `clang-cl`.
   - **Rust:** `rustc`, `cargo`.
3. При отсутствии необходимого инструмента генерирует понятную ошибку `ToolchainError` с рекомендациями по установке.
4. Выполняет компиляцию с учетом соглашений платформы:
   - Для ELF платформ (Linux/Android) передает флаг `-fPIC`.
   - Для MSVC использует флаги `/c`, `/Fo`, `/O2` и архиватор `lib /OUT:`.
   - Для Clang при кросс-компиляции автоматически добавляет `--target=<triple>`.
5. Упаковывает скомпилированные объектные файлы в статический архив:
   - `lib<имя>.a` для Linux, Android и Windows GNU.
   - `<имя>.lib` для Windows MSVC.
6. Автоматически отправляет сборочные директивы через стандартный вывод:
   - `yuki:link-archive=...` — передает путь к архиву линковщику Yukino.
   - `yuki:link-system=stdc++` (или `libcpmt` для MSVC) — подключает рантайм C++ при компиляции C++.
7. Компилятор Yukino перехватывает эти директивы и линкует созданные библиотеки в итоговый бинарный файл.

---

## 3. Структуры ошибок

При возникновении неполадок библиотека вызывает `panic` со строгими типизированными структурами. Это позволяет безопасно перехватывать их через `sus / catch`.

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

Коды ошибок:
- `1001`: Не найден C-компилятор для целевой платформы.
- `1002`: Не найден C++-компилятор для целевой платформы.
- `1003`: Не найдена утилита создания статических архивов (`ar`, `llvm-ar` или `lib`).
- `1004`: Не найден компилятор `rustc`.
- `1005`: Не найден пакетный менеджер `cargo`.
- `2001`: Не указаны исходные файлы для C/C++.
- `2002`: Ошибка компиляции файла C/C++.
- `2003`: Ошибка создания статического архива утилитой `ar` или `lib`.
- `3001`: Ошибка выполнения `cargo build`.
- `3002`: Артефакт сборки Cargo не найден в целевом каталоге.
- `3003`: Ошибка компиляции `rustc`.
- `3004`: Не указан исходный файл Rust или путь к `Cargo.toml`.

---

## 4. Навигация по разделам

- [1. Сборка C и C++ (модуль cc)](1-cc.md)
- [2. Сборка Rust и Cargo (модуль rust)](2-rust.md)

---

[Вперед: Сборка C и C++ (1-cc.md)](1-cc.md)
