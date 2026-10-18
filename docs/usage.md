# Інструкція з запуску AI-агента та локальної збірки проєкту

## 1. Запуск AI-агента

Для ініціалізації та повного розгортання проєкту виконайте наступні кроки в середовищі VS Code:

1. Відкрийте панель Copilot Chat.
2. Викликайте оркестратор `init` за допомогою промпту:
   ```text
   @workspace Прочитай .github/agents/build-engineer.agent.md та виконай команду з .github/prompts/init.prompt.md
   ```

Для виклику окремих команд використовуйте відповідні промпти з каталогу `.github/prompts/`:

- `.github/prompts/git-init.prompt.md` — перевірка та налаштування Git/gitignore.
- `.github/prompts/create-project.prompt.md` — генерація вихідного коду C++.
- `.github/prompts/create-build.prompt.md` — генерація CMakeLists.txt та тестів.
- `.github/prompts/create-actions.prompt.md` — генерація CI workflow.
- `.github/prompts/check.prompt.md` — перевірка стану проєкту.

## 2. Локальне складання та тестування проєкту

Для ручної локальної перевірки збірки та виконання тестів використайте наступні команди в терміналі:

### Швидкий запуск через готовий скрипт
- **Windows (cmd/PowerShell):**
  ```cmd
  ci.bat
  ```
- **Linux / macOS (Bash):**
  ```bash
  chmod +x ci.sh
  ./ci.sh
  ```

### Покрокові команди CMake / CTest
1. **Конфігурація CMake:**
   ```bash
   cmake -S . -B build
   ```
2. **Складання проєкту (генерація виконуваного файлу hello):**
   ```bash
   cmake --build build --config Release
   ```
3. **Запуск виконуваного файлу:**
   - Windows:
     ```powershell
     .\build\Release\hello.exe
     ```
   - Linux / macOS:
     ```bash
     ./build/hello
     ```
4. **Запуск юніт-тестів (BasicAddition):**
   ```bash
   ctest --test-dir build -C Release --output-on-failure
   ```
