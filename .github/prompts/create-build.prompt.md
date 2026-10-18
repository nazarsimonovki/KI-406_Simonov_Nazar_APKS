# Command: create-build

## Передумови
- Команда `create-project` успішно виконана (структура проєкту існує).

## Дії
- Викликати skill `build-and-test`.
- Створити конфігурацію системи складання CMake (`CMakeLists.txt`).
- Згенерувати тестову функцію та юніт-тест (GoogleTest/CTest) у папці `tests/`.

## Очікувані файли
- `CMakeLists.txt`
- `tests/test_main.cpp`

## Команди перевірки
- `cmake -S . -B build` (перевірка валідності синтаксису CMake).

## Повідомлення про помилку
- "Помилка create-build: CMakeLists.txt містить помилки синтаксису, або не вдалося згенерувати тестові файли."