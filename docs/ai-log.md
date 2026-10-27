# Журнал взаємодії з AI-агентом (AI Interaction Log)

Цей документ фіксує послідовність взаємодії з AI-агентом (GitHub Copilot / Antigravity AI) у межах виконання Лабораторної роботи №1 («Проєктування AI-агента для автоматизації розробки та CI/CD», Варіант 1: C++, CMake, GoogleTest/CTest).

---

## 1. Ініціалізація сховища та файлової структури (Скіл `project-scaffold`, команди `/git-init` та `/create-project`)

### Запит (Prompt):
```text
@workspace Створи структуру проєкту для розробки AI-агента відповідно до варіанта 1 (C++, CMake).
Налаштуй .gitignore для виключення каталогу build/, тимчасових файлів, кешу та проміжних файлів компілятора.
Ініціалізуй базовий C++ проєкт: створи каталоги src/, tests/, docs/, файл src/main.cpp із виводом "Hello, World!" та допоміжний файл src/addition.hpp для подальшого тестування.
```

### Відповідь AI та задіяний скіл:
- **Задіяний скіл:** `project-scaffold` (`.github/skills/project-scaffold/SKILL.md`).
- **Створені каталоги:** `src/`, `tests/`, `docs/`.
- **Створені файли:**
  - `src/main.cpp` — точка входу із базовим кодом Hello World.
  - `src/addition.hpp` — заголовний файл із функцією `add(int lhs, int rhs)`.
  - `.gitignore` — правила виключення для `build/`, `.cache/`, `.vs/`, `*.exe`, `*.obj`, `*.o`, `*.pdb`, `*.idb`, `*.ilk`.
  - `README.md` — опис проєкту та варіанта.
  - Команди-промпти: `.github/prompts/git-init.prompt.md`, `.github/prompts/create-project.prompt.md`.

---

## 2. Налаштування системи складання та тестів (Скіл `build-and-test`, команда `/create-build`)

### Запит (Prompt):
```text
@workspace Створи конфігурацію CMake (CMakeLists.txt) та підключи GoogleTest версії v1.14.0 через модуль FetchContent.
Таргет виконуваного файлу має називатися hello.
Підключи CTest, створи тестовий таргет project_tests на базі tests/test_main.cpp та реалізуй юніт-тест BasicAddition для перевірки функції add() із src/addition.hpp.
```

### Відповідь AI та задіяний скіл:
- **Задіяний скіл:** `build-and-test` (`.github/skills/build-and-test/SKILL.md`).
- **Створені файли:**
  - `CMakeLists.txt` — конфігурація збірки стандарту C++17, оголошення таргету `hello`, підключення GoogleTest v1.14.0 через `FetchContent`, створення таргету `project_tests` із підключенням `src/` та лінкуванням `GTest::gtest_main`, автовиявлення тестів через `gtest_discover_tests(project_tests)`.
  - `tests/test_main.cpp` — тестовий сценарій `TEST(BasicAddition, AddsTwoIntegers)`.
  - Команда-промпт: `.github/prompts/create-build.prompt.md`.

---

## 3. Виявлена помилка №1: Невідповідність назви виконуваного файлу вимогам специфікації

### Опис проблеми:
У початковій конфігурації `CMakeLists.txt` було згенеровано таргет:
```cmake
add_executable(hello_world src/main.cpp)
```
Це призвело до генерації файлу `hello_world.exe` замість `hello` (або `hello.exe`), як вимагається у специфікації завдання (перевірка створення виконуваного файлу з точною назвою `hello`).

### Запит на виправлення (Prompt):
```text
У CMakeLists.txt назва таргету виконуваного файлу має бути hello, а не hello_world, щоб генерувався бінарний файл hello (hello.exe). Виправ CMakeLists.txt та онови відповідні інструкції.
```

### Результат виправлення:
Таргет замінено на:
```cmake
add_executable(hello src/main.cpp)
```
Після повторного складання генерується файл `build/Release/hello.exe`, а тести `project_tests.exe` проходять успішно.

---

## 4. Виявлена помилка №2: Потрапляння файлу бази налагодження `tests/vc140.pdb` у Git

### Опис проблеми:
Під час компіляції компілятором MSVC під Windows у каталозі `tests/` було згенеровано налагоджувальний файл `tests/vc140.pdb`, який не було перехоплено початковими правилами `.gitignore`, через що він потрапив до індексу Git. Правила репозиторію суворо забороняють наявність будь-яких проміжних файлів компіляції у контролі версій.

### Запит на виправлення (Prompt):
```text
Видали tests/vc140.pdb з індексу Git і додай шаблони *.pdb, *.idb, *.ilk та каталог .vs/ до .gitignore, щоб проміжні файли компілятора не потрапляли в коміти.
```

### Результат виправлення:
- Виконано `git rm --cached tests/vc140.pdb`.
- До файлу `.gitignore` додано шаблони `*.pdb`, `*.idb`, `*.ilk`, `.vs/`.
- Перевірка через `git status` підтвердила чистоту робочого дерева.

---

## 5. Налаштування CI/CD та артефактів збірки (Скіл `github-actions`, команда `/create-actions`)

### Запит (Prompt):
```text
@workspace Налаштуй мультиплатформний workflow .github/workflows/ci.yml для платформ Ubuntu, Windows та macOS.
Workflow має запускатися на подіях push та pull_request для гілок develop, master та feature/**.
Усі кроки конфігурації, компіляції та тестування мають виконуватися через зовнішні скрипти ci.bat (Windows) та ci.sh (Linux/macOS).
Додай крок actions/upload-artifact@v4 для збереження скомпільованого виконуваного файлу hello як артефакту складання.
```

### Відповідь AI та задіяний скіл:
- **Задіяний скіл:** `github-actions` (`.github/skills/github-actions/SKILL.md`).
- **Створені файли:**
  - `ci.bat` — скрипт автоматизації для Windows (`cmake -S . -B build`, `cmake --build build --config Release`, `ctest --test-dir build -C Release --output-on-failure`).
  - `ci.sh` — скрипт автоматизації для Unix (`set -euo pipefail`, `cmake -S . -B build -DCMAKE_BUILD_TYPE=Release`, `cmake --build build --config Release`, `ctest --test-dir build -C Release --output-on-failure`).
  - `.github/workflows/ci.yml` — workflow з матрицею `[ubuntu-latest, windows-latest, macos-latest]`, запуском скриптів та збереженням артефакту `hello`.
  - Команда-промпт: `.github/prompts/create-actions.prompt.md`.

---

## 6. Перехід до інтерактивних слеш-команд та строга типізація інструкцій

### Запит (Prompt):
```text
Перепиши всі інструкції для агента, щоб вони стали інтерактивними слеш-командами (/init, /git-init, /create-project, /create-build, /create-actions, /check), які автоматично з'являються у випадаючому меню при введенні / у GitHub Copilot Chat.
Перепиши інструкції у строгий технічний стиль без емодзі та зайвої інформації, з точним зазначенням назв файлів і папок.
```

### Результат виконання:
- До кожного файлу `.prompt.md` у каталозі `.github/prompts/` додано метадані YAML frontmatter (`description: "..."`), що забезпечує автоматичну інтеграцію у меню швидкого вибору Copilot Chat.
- Оновлено специфікації скілів у `.github/skills/`:
  - `project-scaffold/SKILL.md`
  - `build-and-test/SKILL.md`
  - `github-actions/SKILL.md`
- Зафіксовано строгу структуру опису: призначення, вхідні дані, цільові каталоги й файли, кроки виконання, критерії верифікації, регламент помилок.

---

## 7. Виявлена помилка №3: Системна колізія команди `/init` у GitHub Copilot Chat

### Опис проблеми:
При введенні користувачем команди `/init` у вікні GitHub Copilot Chat середовище перехопило команду як власну системну дію Copilot (генерація інструкцій для робочого простору) і без попередження створило зайвий файл `.github/copilot-instructions.md`, замість виклику файлу `.github/prompts/init.prompt.md`.

### Дії щодо усунення:
- Зайвий згенерований файл `.github/copilot-instructions.md` видалено з проєкту.
- У документації (`README.md`, `docs/usage.md`) зафіксовано рекомендацію щодо виклику оркестратора за явним посиланням на маніфест:
  ```text
  @workspace Прочитай .github/agents/build-engineer.agent.md та виконай .github/prompts/init.prompt.md
  ```
  або використання окремих автономних команд (`/git-init`, `/create-project`, `/create-build`, `/create-actions`, `/check`), які не мають системних колізій із Copilot.

---

## 8. Результати повної верифікації (Команда `/check`)

Команда `/check` виконала комплексну перевірку проєкту:
- **Структура каталогів та файлів:** повна відповідність вимогам (`.github/agents/`, `.github/prompts/`, `.github/skills/`, `.github/workflows/`, `src/`, `tests/`, `docs/`).
- **Локальна збірка:** успішна через запуск `ci.bat`.
  ```text
  hello.vcxproj -> N:\LABS\APKS\LAB1\build\Release\hello.exe
  project_tests.vcxproj -> N:\LABS\APKS\LAB1\build\Release\project_tests.exe
  1/1 Test #1: BasicAddition.AddsTwoIntegers ....   Passed    0.01 sec
  100% tests passed, 0 tests failed out of 1
  ```
- **Стан репозиторію Git:** робоче дерево чисте, тимчасові файли та бінарні артефакти відсутні в індексі.
