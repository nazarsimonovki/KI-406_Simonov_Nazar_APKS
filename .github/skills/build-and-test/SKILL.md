# Skill: build-and-test

## Призначення
Генерація конфігурації системи складання CMake та створення модульного тесту на базі GoogleTest і CTest.

## Вхідні дані
- Наявність директорії `src/` із файлами `main.cpp` та `addition.hpp`.
- Фреймворк тестування: GoogleTest (реліз v1.14.0) через CMake FetchContent.
- Диспетчер тестів: CTest.

## Цільові файли та каталоги
- Каталог `tests/`
- Файл `CMakeLists.txt`
- Файл `tests/test_main.cpp`

## Дії
1. Створити у корені проєкту файл `CMakeLists.txt`:
   - Встановити стандарт C++17.
   - Оголосити таргет `add_executable(hello src/main.cpp)`.
   - Активувати тестування `include(CTest)`.
   - Підключити GoogleTest v1.14.0 через модуль `FetchContent`.
   - Оголосити тестовий таргет `add_executable(project_tests tests/test_main.cpp)`.
   - Налаштувати `target_include_directories(project_tests PRIVATE src)`.
   - Налаштувати `target_link_libraries(project_tests PRIVATE GTest::gtest_main)`.
   - Зареєструвати тести через `gtest_discover_tests(project_tests)`.
2. Створити файл `tests/test_main.cpp` із перевіркою коректності результату функції `add(2, 3) == 5`.

## Критерії верифікації
- Успішна конфігурація: `cmake -S . -B build` (код повернення 0).
- Успішне складання: `cmake --build build --config Release` (код повернення 0).
- Успішний запуск тестів: `ctest --test-dir build -C Release --output-on-failure` (100% тестів пройдено).

## Обробка помилок
- Помилка: збій завантаження або лінкування GoogleTest, синтаксична помилка в CMakeLists.txt або збій компіляції тестів.
- Дія: вивести повний лог компілятора/CMake та перервати процес оркестрації.
