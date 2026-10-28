# Журнал взаємодії з AI-агентом (AI Interaction Log)

Цей документ фіксує хронологічну послідовність взаємодії з AI-агентом (GitHub Copilot / Antigravity AI) у межах виконання Лабораторної роботи №1 («Проєктування AI-агента для автоматизації розробки та CI/CD», Варіант 1: C++, CMake, GoogleTest/CTest).

---

## 1. Початковий запит та ініціалізація структури (Скіл `@project-scaffold`, команди `@git-init.prompt.md` та `@create-project.prompt.md`)

### Запит (Prompt):
```text
@workspace Створи структуру проєкту для розробки AI-агента відповідно до мови програмування C++, системи збірки CMake.
Налаштуй .gitignore для виключення каталогу build/, тимчасових файлів, кешу та проміжних файлів компілятора.
Ініціалізуй базовий C++ проєкт: створи каталоги src/, tests/, docs/, файл src/main.cpp із виводом "Hello, World!" та допоміжний файл src/addition.hpp для подальшого тестування.
```

### Відповідь AI та задіяний скіл:
- **Задіяний скіл:** `@project-scaffold` (`.github/skills/project-scaffold/SKILL.md`).
- **Створені каталоги:** `src/`, `tests/`, `docs/`.
- **Створені файли:**
  - `src/main.cpp` — точка входу застосунку з базовим кодом Hello World.
  - `src/addition.hpp` — заголовний файл із тестовою функцією `constexpr int add(int lhs, int rhs)`.
  - `.gitignore` — правила виключення для директорії `build/`, системних кешів, тимчасових файлів IDE та об'єктних файлів.
  - `README.md` — опис проєкту, варіанта та правил запуску.
  - Команди-промпти: `.github/prompts/git-init.prompt.md`, `.github/prompts/create-project.prompt.md`.

---

## 2. Налаштування системи складання та тестів (Скіл `@build-and-test`, команда `@create-build.prompt.md`)

### Запит (Prompt):
```text
@workspace Створи конфігурацію CMake (CMakeLists.txt) та підключи GoogleTest версії v1.14.0 через модуль FetchContent.
Таргет виконуваного файлу має називатися hello.
Підключи CTest, створи тестовий таргет project_tests на базі tests/test_main.cpp та реалізуй юніт-тест BasicAddition для перевірки функції add() із src/addition.hpp.
```

### Відповідь AI та задіяний скіл:
- **Задіяний скіл:** `@build-and-test` (`.github/skills/build-and-test/SKILL.md`).
- **Створені файли:**
  - `CMakeLists.txt` — конфігурація збірки стандарту C++17, створення таргету `hello`, підключення GoogleTest v1.14.0 через модуль `FetchContent`, створення таргету `project_tests` із підключенням `src/` та лінкуванням `GTest::gtest_main`, автовиявлення тестів через `gtest_discover_tests(project_tests)`.
  - `tests/test_main.cpp` — тестовий сценарій `TEST(BasicAddition, AddsTwoIntegers)`, що перевіряє `add(2, 3) == 5`.
  - Команда-промпт: `.github/prompts/create-build.prompt.md`.

---

## 3. Виявлена помилка №1: Невідповідність назви виконуваного файлу вимогам специфікації

### Опис проблеми:
У початковій конфігурації `CMakeLists.txt` було згенеровано таргет:
```cmake
add_executable(hello_world src/main.cpp)
```
Це призвело до генерації файлу `hello_world.exe` замість `hello` (або `hello.exe`), як вимагається у специфікації завдання (перевірка створення виконуваного файлу з назвою `hello`).

### Запит на виправлення (Prompt):
```text
У CMakeLists.txt назва таргету виконуваного файлу має бути саме hello, а не hello_world, щоб генерувався бінарний файл hello.exe. Виправ CMakeLists.txt та онови відповідні інструкції.
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
Під час компіляції компілятором MSVC під Windows у каталозі `tests/` було згенеровано налагоджувальний файл `tests/vc140.pdb`, який не було перехоплено початковими правилами `.gitignore`, через що він потрапив до індексу Git. Правила репозиторію суворо забороняють наявність будь-яких проміжних файлів компіляції у версійному контролі.

### Запит на виправлення (Prompt):
```text
Видали tests/vc140.pdb з індексу Git і додай шаблони *.pdb, *.idb, *.ilk та каталог .vs/ до .gitignore, щоб проміжні файли компілятора більше не потрапляли в коміти.
```

### Результат виправлення:
- Виконано вилучення з індексу: `git rm --cached tests/vc140.pdb`.
- До файлу `.gitignore` додано шаблони `*.pdb`, `*.idb`, `*.ilk`, `.vs/`.
- Перевірка через `git status` підтвердила повну чистоту репозиторію.

---

## 5. Налаштування CI/CD та завантаження артефактів (Скіл `@github-actions`, команда `@create-actions.prompt.md`)

### Запит (Prompt):
```text
@workspace Налаштуй мультиплатформний workflow .github/workflows/ci.yml для платформ Ubuntu, Windows та macOS.
Workflow має запускатися на подіях push та pull_request для гілок develop, master та feature/**.
Усі кроки конфігурації, компіляції та тестування мають виконуватися через зовнішні скрипти ci.bat (Windows) та ci.sh (Linux/macOS).
Додай крок actions/upload-artifact@v4 для збереження скомпільованого виконуваного файлу hello як артефакту складання.
```

### Відповідь AI та задіяний скіл:
- **Задіяний скіл:** `@github-actions` (`.github/skills/github-actions/SKILL.md`).
- **Створені файли:**
  - `ci.bat` — скрипт автоматизації для Windows (`cmake -S . -B build`, `cmake --build build --config Release`, `ctest --test-dir build -C Release --output-on-failure`).
  - `ci.sh` — скрипт автоматизації для Unix (`set -euo pipefail`, `cmake -S . -B build -DCMAKE_BUILD_TYPE=Release`, `cmake --build build --config Release`, `ctest --test-dir build -C Release --output-on-failure`).
  - `.github/workflows/ci.yml` — workflow з матрицею `[ubuntu-latest, windows-latest, macos-latest]`, викликом скриптів та збереженням артефакту `hello`.
  - Команда-промпт: `.github/prompts/create-actions.prompt.md`.

---

## 6. Виявлена помилка №3: Системна колізія вбудованої команди `/init` у Copilot Chat

### Опис проблеми:
При спробі викликати оркестратор через команду `/init` у Copilot Chat, середовище перехопило введення як внутрішню зарезервовану команду самого GitHub Copilot (генерація інструкцій для репозиторію) і автоматично створило сторонній небажаний файл `.github/copilot-instructions.md`, проігнорувавши наш файл інструкцій.

### Запит на виправлення (Prompt):
```text
Очисть проєкт від того, що помилково створив або змінив Copilot.
```

### Результат виправлення:
- Файл `.github/copilot-instructions.md` повністю видалено з проєкту.
- Проведено аналіз колізій із системними командами середовища VS Code Copilot.

---

## 7. Перехід від слеш-команд до виклику через символ `@` (равлик)

### Запит (Prompt):
```text
Перепиши всі інструкції для агента, щоб запуск команд відбувався не зі слеша, а через символ @ (равлик) і назву файлу інструкції (@init.prompt.md, @git-init.prompt.md, @create-project.prompt.md, @create-build.prompt.md, @create-actions.prompt.md, @check.prompt.md) або навички (@project-scaffold тощо).
Перепиши інструкції у строгий технічний стиль без емодзі та зайвої інформації, з точним зазначенням назв файлів і папок.
```

### Результат виконання:
- Виключено залежність від слешів (`/`), що гарантує відсутність колізій із вбудованими функціями Copilot.
- Оновлено документацію (`README.md`, `docs/usage.md`, `docs/architecture.md`), маніфест (`.github/agents/build-engineer.agent.md`) та всі файли промптів у `.github/prompts/`:
  - `@init.prompt.md` — головний оркестратор
  - `@git-init.prompt.md` — налаштування Git та `.gitignore`
  - `@create-project.prompt.md` — генерація коду та каталогів
  - `@create-build.prompt.md` — створення `CMakeLists.txt` та тестів
  - `@create-actions.prompt.md` — створення CI/CD пайплайну
  - `@check.prompt.md` — комплексна верифікація
- Навички у `.github/skills/` отримали строгу стандартизовану структуру: призначення, вхідні дані, цільові каталоги та файли, кроки виконання, критерії верифікації, обробка помилок.

---

## 8. Очищення робочого дерева та повне тестування оркестратора з нуля

### Запит (Prompt):
```text
Очисти проєкт від згенерованих файлів, щоб можна було з нуля запустити оркестратор і перевірити його роботу.
```

### Дії щодо підготовки та перевірки:
1. Усі актуальні налаштування агента, інструкції та документація зафіксовані в коміті Git (`a0b130a`).
2. Робочий простір очищено від раніше згенерованих цільових артефактів:
   - Видалено директорію збірки `build/`.
   - Видалено каталоги `src/` та `tests/`.
   - Видалено файли `CMakeLists.txt`, `ci.bat`, `ci.sh`, `.github/workflows/ci.yml`, `.gitignore`.
3. У репозиторії залишено виключно ядро агента: маніфест, промпти, навички та документацію.
4. У Copilot Chat надіслано команду запуску оркестратора:
   ```text
   @workspace Прочитай .github/agents/build-engineer.agent.md та виконай @init.prompt.md
   ```
5. **Результат тестування:** AI-агент в автономному режимі з нуля успішно відтворив усю структуру: створив каталоги `src/` і `tests/`, записав `src/main.cpp` та `src/addition.hpp`, створив конфігурацію `CMakeLists.txt` (таргети `hello`, `project_tests`), згенерував скрипти `ci.bat`/`ci.sh` та workflow `.github/workflows/ci.yml`.

---

## 9. Результати фінальної верифікації проєкту (Команда `@check.prompt.md`)

Запуск інструкції `@check.prompt.md` та локального скрипта перевірки `ci.bat`:
- **Файлова структура:** усі обов'язкові каталоги та файли присутні.
- **Локальна збірка:** компілятор MSVC успішно зібрав проєкт у конфігурації Release без попереджень і помилок:
  ```text
  hello.vcxproj -> N:\LABS\APKS\LAB1\build\Release\hello.exe
  project_tests.vcxproj -> N:\LABS\APKS\LAB1\build\Release\project_tests.exe
  ```
- **Модульне тестування CTest:**
  ```text
  Test project N:/LABS/APKS/LAB1/build
      Start 1: BasicAddition.AddsTwoIntegers
  1/1 Test #1: BasicAddition.AddsTwoIntegers ....   Passed    0.02 sec

  100% tests passed, 0 tests failed out of 1
  Total Test time (real) =   0.03 sec
  ```
- **Стан репозиторію Git:** усі артефакти складання ізольовано через `.gitignore`, робоче дерево чисте.
