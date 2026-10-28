---
description: "Генерація CMakeLists.txt та налаштування юніт-тестів GoogleTest/CTest"
---

# @create-build.prompt.md

## Передумови
- Успішне виконання інструкції `@create-project.prompt.md`.
- Наявність файлів `src/main.cpp` та `src/addition.hpp`.
- Наявність навички `@build-and-test` (`.github/skills/build-and-test/SKILL.md`).

## Цільові файли та каталоги
- Файл `CMakeLists.txt`
- Файл `tests/test_main.cpp`

## Дії
1. Створити у корені проєкту файл `CMakeLists.txt` із такими параметрами:
   - Мінімальна версія CMake: `cmake_minimum_required(VERSION 3.20)`
   - Назва проєкту: `project(HelloWorldApp LANGUAGES CXX)`
   - Стандарт мови: `set(CMAKE_CXX_STANDARD 17)`, `set(CMAKE_CXX_STANDARD_REQUIRED ON)`, `set(CMAKE_CXX_EXTENSIONS OFF)`
   - Підключення модуля `include(CTest)`
   - Таргет виконуваного файлу з назвою `hello` на основі `src/main.cpp`
   - Блок `if(BUILD_TESTING)`: підключення `FetchContent`, завантаження `googletest` v1.14.0, створення таргету `project_tests` із вихідним файлом `tests/test_main.cpp`, лінкування з `GTest::gtest_main`, підключення каталогу `src`, реєстрація через `include(GoogleTest)` та `gtest_discover_tests(project_tests)`
2. Створити файл `tests/test_main.cpp`:
   ```cpp
   #include "addition.hpp"

   #include <gtest/gtest.h>

   TEST(BasicAddition, AddsTwoIntegers) {
       EXPECT_EQ(add(2, 3), 5);
   }
   ```

## Очікуваний результат
- Валідний файл `CMakeLists.txt` у корені проєкту.
- Файл `tests/test_main.cpp` із тестом `BasicAddition`.

## Команди верифікації
- `cmake -S . -B build`
- `cmake --build build --config Release`
- `ctest --test-dir build -C Release --output-on-failure`

## Обробка помилок
- У разі помилки вивести: "Помилка @create-build.prompt.md: синтаксична помилка у CMakeLists.txt або збій компіляції таргету project_tests."
