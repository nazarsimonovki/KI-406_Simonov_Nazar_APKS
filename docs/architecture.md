# Архітектура AI-агента (Build Engineer)

Цей документ визначає технічну архітектуру, модульну структуру та протокол взаємодії компонентів AI-агента для автоматизації збірки й тестування кросплатформного проєкту на C++.

## 1. Задачі, які вирішує агент
- Автоматизація створення стандартизованої структури каталогів та файлів вихідного коду C++17.
- Усунення ручної конфігурації мультиплатформної збірки через CMake.
- Інтеграція модульного тестування GoogleTest із реєстрацією тестів у системі CTest.
- Формування CI/CD конвеєра в GitHub Actions із матрицею трьох цільових ОС (Ubuntu, Windows, macOS).
- Забезпечення чистоти репозиторію через ізоляцію проміжних артефактів компіляції та заборону збереження секретів.

## 2. Модульні навички (Skills)
Навички є функціональними блоками для виконання конкретних операцій:
- `project-scaffold` (`.github/skills/project-scaffold/SKILL.md`): створення каталогів `src/`, `tests/`, `docs/`, вихідного файлу `src/main.cpp`, допоміжного файлу `src/addition.hpp`, `README.md` та конфігурації `.gitignore`.
- `build-and-test` (`.github/skills/build-and-test/SKILL.md`): формування кореневого `CMakeLists.txt` та модульного тесту `tests/test_main.cpp` з використанням GoogleTest v1.14.0.
- `github-actions` (`.github/skills/github-actions/SKILL.md`): створення файлу робочого процесу `.github/workflows/ci.yml` та скриптів автоматизації `ci.bat` і `ci.sh`.

## 3. Інтерактивні слеш-команди (Commands)
Команди реалізовані у вигляді файлів `.prompt.md` із метаданими YAML frontmatter (`description: "..."`), що забезпечує їх автоматичну появу у списку швидкого вибору GitHub Copilot Chat при введенні `/`:
- `/git-init` (`.github/prompts/git-init.prompt.md`): перевірка репозиторію, запуск `git init`, оновлення `.gitignore`.
- `/create-project` (`.github/prompts/create-project.prompt.md`): виклик навички `project-scaffold`.
- `/create-build` (`.github/prompts/create-build.prompt.md`): виклик навички `build-and-test`.
- `/create-actions` (`.github/prompts/create-actions.prompt.md`): виклик навички `github-actions`.
- `/check` (`.github/prompts/check.prompt.md`): статична перевірка наявності файлів, аналіз індексу Git, повна збірка та виконання тестів.
- `/init` (`.github/prompts/init.prompt.md`): оркестратор, що послідовно запускає повний ланцюжок команд (`/git-init` -> `/create-project` -> `/create-build` -> `/create-actions` -> `/check`).

## 4. Потоки даних та артефакти між етапами
1. Від `/git-init` до `/create-project`: передається ініціалізоване робоче дерево Git із налаштованими правилами ігнорування.
2. Від `/create-project` до `/create-build`: передається структура каталогів та наявність файлів `src/main.cpp` і `src/addition.hpp`.
3. Від `/create-build` до `/create-actions`: передаються параметри збірки (CMakeLists.txt, таргет `hello`, таргет `project_tests`, CTest), на основі яких формуються скрипти `ci.bat`, `ci.sh` та workflow `ci.yml`.
4. Від усіх етапів до `/check`: передається весь сформований комплекс файлів для підсумкової перевірки.

## 5. Обробка помилок оркестратором
Команда-оркестратор `/init` керується такими регламентами:
1. Строга послідовність: виконання кроку `N+1` можливе лише за умови успішного виконання кроку `N` (код повернення 0).
2. Миттєва зупинка: при виникненні будь-якого збою конвеєр негайно зупиняється.
3. Інформативність: виводиться назва команди, на якій стався збій, та код помилки або лог збою. Приховування або ігнорування помилок заборонено.
4. Ідемпотентність: повторний виклик команди `/init` не знищує коректні файли, а доповнює або виправляє пошкоджені елементи.

## 6. Фізична структура репозиторію
```text
.
├── .github/
│   ├── agents/
│   │   └── build-engineer.agent.md   # Маніфест агента
│   ├── prompts/                      # Інтерактивні слеш-команди Copilot Chat
│   │   ├── check.prompt.md           # Команда /check
│   │   ├── create-actions.prompt.md  # Команда /create-actions
│   │   ├── create-build.prompt.md    # Команда /create-build
│   │   ├── create-project.prompt.md  # Команда /create-project
│   │   ├── git-init.prompt.md        # Команда /git-init
│   │   └── init.prompt.md            # Команда /init (Оркестратор)
│   ├── skills/                       # Модульні навички агента
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
