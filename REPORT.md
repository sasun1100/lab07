# Отчёт по лабораторной работе №7

**Студент:** Maksim Alehanov
**Группа:** ИУ8-25
**Тема:** Изучение систем управления пакетами на примере Hunter

---

## 1. Цель работы

Подключить зависимости через менеджер пакетов Hunter, заменив ручное
подключение GTest (git submodule) на пакет из Hunter. Проект — библиотека
`print`.

## 2. Выполнение

### 2.1. Подготовка проекта

Проект скопирован из лабораторной работы №6:

```sh
git clone https://github.com/sasun1100/lab06 lab07
cd lab07
git remote set-url origin https://github.com/sasun1100/lab07
```

### 2.2. Подключение Hunter

Скачан `HunterGate.cmake` и добавлен вызов в начало `CMakeLists.txt`:

```cmake
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.26.9.tar.gz"
    SHA1 "a962c9f0cfd6f8fc62ea25943d89debfa61d4d4a"
    LOCAL
)
```

### 2.3. GTest через Hunter

Удалён submodule `third-party/gtest`. Подключение GTest теперь:

```cmake
hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

add_executable(check tests/test_print.cpp)
target_link_libraries(check print GTest::gtest_main)
```

Версия GTest зафиксирована в `cmake/Hunter/config.cmake`:

```cmake
hunter_config(GTest VERSION 1.12.1)
```

### 2.4. Демонстрационное приложение

Добавлен таргет `demo` (`demo/main.cpp`), использующий библиотеку `print`:
читает слова со стандартного ввода и пишет их в файл из переменной `LOG_PATH`.

```cpp
#include <print.hpp>
...
const char* log_path = std::getenv("LOG_PATH");
std::string text;
while (std::cin >> text) {
  std::ofstream out{log_path, std::ios_base::app};
  print(text, out);
  out << std::endl;
}
```

### 2.5. Сборка

```sh
$ cmake -H. -B_builds -DBUILD_TESTS=ON
$ cmake --build _builds
$ ctest --test-dir _builds --output-on-failure
100% tests passed, 0 tests failed out of 1
```

Hunter при первой сборке скачивает и собирает GTest в кэш `~/.hunter`.

Проверка demo:

```sh
$ export LOG_PATH=log.txt
$ printf "alpha\nbeta\ngamma\n" | ./_builds/demo
$ cat log.txt
alpha
beta
gamma
```

## 3. Результаты

- GTest подключается через Hunter вместо git submodule.
- Сборка и тесты проходят (CI на GitHub Actions с кэшированием `~/.hunter`).
- Демо-приложение использует библиотеку `print`.
