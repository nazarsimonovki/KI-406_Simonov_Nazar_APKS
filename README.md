# Лабораторна робота № 1: Проєктування AI-агента для автоматизації розробки та CI/CD

## Інформація про студента
- **ПІБ:** Сімонов Назар Андрійович
- **Група:** КІ-406
- **Варіант:** 1 (C++, CMake, GoogleTest/CTest)

## Опис проєкту
Спеціалізований AI-агент (Build Engineer) для повної автоматизації життєвого циклу розробки кросплатформного застосунку "Hello World" мовою C++17 із системою складання CMake, тестуванням через GoogleTest/CTest та CI/CD конвеєром у GitHub Actions.

Характеристики проєкту:
- Мова програмування: C++17
- Система складання: CMake (мінімальна версія 3.20)
- Модульне тестування: GoogleTest v1.14.0 (підключення через FetchContent), CTest
- CI/CD конвеєр: GitHub Actions (матриця Ubuntu, Windows, macOS)

## Команди та навички агента (виклик через @)

Взаємодія з агентом у GitHub Copilot Chat у середовищі VS Code здійснюється не через слеш-команди, а через механізм згадування файлів та навичок за допомогою символу `@` (равлик) і назви відповідного файлу промпту або навички з директорії `.github/`.

### Доступні команди агента:
- `@init.prompt.md` — головний оркестратор: послідовний запуск усього ланцюжка розгортання та тестування проєкту (`.github/prompts/init.prompt.md`).
- `@git-init.prompt.md` — ініціалізація локального репозиторію Git та налаштування файлу `.gitignore` (`.github/prompts/git-init.prompt.md`).
- `@create-project.prompt.md` — створення структури каталогів (`src/`, `tests/`, `docs/`) та вихідних файлів (`src/main.cpp`, `src/addition.hpp`) через навичку `@project-scaffold` (`.github/prompts/create-project.prompt.md`).
- `@create-build.prompt.md` — генерація конфігураційного файлу `CMakeLists.txt` та модульного тесту `tests/test_main.cpp` через навичку `@build-and-test` (`.github/prompts/create-build.prompt.md`).
- `@create-actions.prompt.md` — створення файлу workflow `.github/workflows/ci.yml` та скриптів автоматизації `ci.bat` і `ci.sh` через навичку `@github-actions` (`.github/prompts/create-actions.prompt.md`).
- `@check.prompt.md` — статична та динамічна перевірка коректності структури файлів, компіляції та виконання тестів (`.github/prompts/check.prompt.md`).

### Модульні навички (Skills):
- `@project-scaffold` (`.github/skills/project-scaffold/SKILL.md`)
- `@build-and-test` (`.github/skills/build-and-test/SKILL.md`)
- `@github-actions` (`.github/skills/github-actions/SKILL.md`)

### Інструкція з виклику

1. Відкрити репозиторій у середовищі VS Code.
2. Відкрити панель GitHub Copilot Chat (`Ctrl+Alt+I` або `Ctrl+Shift+I`).
3. Для повного розгортання проєкту звернутися до файлу оркестратора через `@` (равлик):
   ```text
   @init.prompt.md
   ```
   або з явним контекстом робочого простору:
   ```text
   @workspace Прочитай .github/agents/build-engineer.agent.md та виконай @init.prompt.md
   ```
4. Для виконання окремого етапу введіть `@` та вкажіть назву відповідного файлу команди, наприклад:
   ```text
   @create-project.prompt.md
   ```

## Локальна збірка та запуск тестів

Вимоги до оточення: CMake 3.20+, компілятор C++17 (MSVC / GCC / Clang).

Швидкий запуск через скрипти автоматизації:
- Windows:
  ```cmd
  ci.bat
  ```
- Linux / macOS:
  ```bash
  chmod +x ci.sh
  ./ci.sh
  ```

Покрокове виконання:
```powershell
cmake -S . -B build
cmake --build build --config Release
ctest --test-dir build -C Release --output-on-failure
```

Виконуваний файл застосунку: `build/Release/hello.exe` (Windows) або `build/hello` (Unix).
