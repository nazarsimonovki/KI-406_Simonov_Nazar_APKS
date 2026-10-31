# Архітектура AI-агента (Build Engineer)

Цей документ визначає технічну архітектуру, модульну структуру та протокол взаємодії компонентів AI-агента для автоматизації збірки й тестування кросплатформного проєкту на C++.

## 1. Задачі, які вирішує агент
- Автоматизація створення стандартизованої структури каталогів та файлів вихідного коду C++17.
- Усунення ручної конфігурації мультиплатформної збірки через CMake.
- Інтеграція модульного тестування GoogleTest із реєстрацією тестів у системі CTest.
- Формування CI/CD конвеєра в GitHub Actions із матрицею трьох цільових ОС (Ubuntu, Windows, macOS).
- Забезпечення чистоти репозиторію через ізоляцію проміжних артефактів компіляції та заборону збереження секретів.

## 2. Модульні навички (Skills)
Навички є функціональними блоками для виконання конкретних операцій (можуть викликатися через `@` із зазначенням назви навички або шляху до файлу):
- `project-scaffold` (`.github/skills/project-scaffold/SKILL.md`): створення каталогів `src/`, `tests/`, `docs/`, вихідного файлу `src/main.cpp`, допоміжного файлу `src/addition.hpp`, `README.md` та конфігурації `.gitignore`.
- `build-and-test` (`.github/skills/build-and-test/SKILL.md`): формування кореневого `CMakeLists.txt` та модульного тесту `tests/test_main.cpp` з використанням GoogleTest v1.14.0.
- `github-actions` (`.github/skills/github-actions/SKILL.md`): створення файлу робочого процесу `.github/workflows/ci.yml` та скриптів автоматизації `ci.bat` і `ci.sh`.

## 3. Команди агента (виклик через @)
Запуск інструкцій користувачем у середовищі Copilot Chat відбувається не через системні слеш-команди (`/`), а через механізм згадування файлів через символ `@` (равлик) і назву файлу команди:
- `@git-init.prompt.md` (`.github/prompts/git-init.prompt.md`): перевірка репозиторію, запуск `git init`, оновлення `.gitignore`.
- `@create-project.prompt.md` (`.github/prompts/create-project.prompt.md`): виклик навички `@project-scaffold`.
- `@create-build.prompt.md` (`.github/prompts/create-build.prompt.md`): виклик навички `@build-and-test`.
- `@create-actions.prompt.md` (`.github/prompts/create-actions.prompt.md`): виклик навички `@github-actions`.
- `@check.prompt.md` (`.github/prompts/check.prompt.md`): статична перевірка наявності файлів, аналіз індексу Git, повна збірка та виконання тестів.
- `@init.prompt.md` (`.github/prompts/init.prompt.md`): оркестратор, що послідовно викликає повний конвеєр інструкцій (`@git-init.prompt.md` -> `@create-project.prompt.md` -> `@create-build.prompt.md` -> `@create-actions.prompt.md` -> `@check.prompt.md`).

## 4. Потоки даних та артефакти між етапами
1. Від `@git-init.prompt.md` до `@create-project.prompt.md`: передається ініціалізоване робоче дерево Git із налаштованими правилами ігнорування.
2. Від `@create-project.prompt.md` до `@create-build.prompt.md`: передається структура каталогів та наявність файлів `src/main.cpp` і `src/addition.hpp`.
3. Від `@create-build.prompt.md` до `@create-actions.prompt.md`: передаються параметри збірки (CMakeLists.txt, таргет `hello`, таргет `project_tests`, CTest), на основі яких формуються скрипти `ci.bat`, `ci.sh` та workflow `ci.yml`.
4. Від усіх етапів до `@check.prompt.md`: передається весь сформований комплекс файлів для підсумкової перевірки.

## 5. Обробка помилок оркестратором
Команда-оркестратор `@init.prompt.md` керується такими регламентами:
1. Строга послідовність: виконання кроку `N+1` можливе лише за умови успішного виконання кроку `N` (код повернення 0).
2. Миттєва зупинка: при виникненні будь-якого збою конвеєр негайно зупиняється.
3. Інформативність: виводиться назва файлу команди, на якій стався збій, та код помилки або лог збою. Приховування або ігнорування помилок заборонено.
4. Ідемпотентність: повторний виклик команди `@init.prompt.md` не знищує коректні файли, а доповнює або виправляє пошкоджені елементи.

## 6. Фізична структура репозиторію
```text
.
├── .github/
│   ├── agents/
│   │   └── build-engineer.agent.md   # Маніфест агента
│   ├── prompts/                      # Команди агента (виклик через @)
│   │   ├── check.prompt.md           # Інструкція @check.prompt.md
│   │   ├── create-actions.prompt.md  # Інструкція @create-actions.prompt.md
│   │   ├── create-build.prompt.md    # Інструкція @create-build.prompt.md
│   │   ├── create-project.prompt.md  # Інструкція @create-project.prompt.md
│   │   ├── git-init.prompt.md        # Інструкція @git-init.prompt.md
│   │   └── init.prompt.md            # Інструкція @init.prompt.md (Оркестратор)
│   ├── skills/                       # Модульні навички агента (виклик через @)
│   │   ├── build-and-test/
│   │   │   └── SKILL.md
│   │   ├── github-actions/
│   │   │   └── SKILL.md
│   │   └── project-scaffold/
│   │       └── SKILL.md
│   └── workflows/
│       └── ci.yml                    # Мультиплатформний workflow GitHub Actions
├── docs/                             # Документація проєкту
│   ├── ai-log.md                     # Журнал взаємодії з агентом
│   ├── architecture.md               # Опис архітектури агента
│   └── usage.md                      # Інструкція користувача
├── src/                              # Вихідний код C++
│   ├── addition.hpp
│   └── main.cpp
├── tests/                            # Модульні тести GoogleTest
│   └── test_main.cpp
├── CMakeLists.txt                    # Конфігурація збірки CMake
├── ci.bat                            # Скрипт автоматизації для Windows
├── ci.sh                             # Скрипт автоматизації для Linux/macOS
├── README.md                         # Головний опис проєкту
└── .gitignore                        # Правила ігнорування тимчасових файлів
```
