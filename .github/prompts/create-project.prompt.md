---
description: "Генерація структури каталогів та вихідного коду C++ Hello World"
---

# @create-project.prompt.md

## Передумови
- Успішне виконання інструкції `@git-init.prompt.md`.
- Наявність маніфесту `.github/agents/build-engineer.agent.md` та навички `@project-scaffold` (`.github/skills/project-scaffold/SKILL.md`).

## Цільові файли та каталоги
- Каталог `src/`
- Каталог `tests/`
- Каталог `docs/`
- Файл `src/main.cpp`
- Файл `src/addition.hpp`
- Файл `README.md`

## Дії
1. Створити каталоги `src/`, `tests/` та `docs/` у корені проєкту (якщо вони відсутні).
2. Створити файл `src/main.cpp` із точним вмістом:
   ```cpp
   #include <iostream>

   int main() {
       std::cout << "Hello, World!" << std::endl;
       return 0;
   }
   ```
3. Створити заголовний файл `src/addition.hpp` для наступного етапу тестування:
   ```cpp
   #pragma once

   constexpr int add(int lhs, int rhs) {
       return lhs + rhs;
   }
   ```
4. Створити або оновити файл `README.md` з описом проєкту, структури та варіанта.

## Очікуваний результат
- Створено каталоги `src/`, `tests/`, `docs/`.
- Створено файли `src/main.cpp` та `src/addition.hpp`.
- Створено файл `README.md`.

## Команди верифікації
- Перевірка наявності файлів `src/main.cpp`, `src/addition.hpp` та каталогів `tests/`, `docs/`.

## Обробка помилок
- У разі відсутності будь-якого з цільових файлів вивести: "Помилка @create-project.prompt.md: не вдалося створити вихідні файли коду або файлову структуру проєкту."
