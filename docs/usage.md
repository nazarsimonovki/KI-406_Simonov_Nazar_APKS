# Інструкція з використання слеш-команд та збірки проєкту

## 1. Робота зі слеш-командами в GitHub Copilot Chat

Усі команди агента реалізовані як інтерактивні слеш-команди через prompt-файли формату `.github/prompts/<назва>.prompt.md`. Вони автоматично реєструються середовищем VS Code та відображаються у випадаючому списку автодоповнення при введенні символу `/` у чаті.

### Порядок роботи:
1. Відкрити кореневу директорію репозиторію у VS Code.
2. Відкрити панель GitHub Copilot Chat.
3. Ввести `/` у полі введення та обрати цільову команду:

| Слеш-команда | Файл конфігурації | Опис та призначення |
| :--- | :--- | :--- |
| `/init` | `.github/prompts/init.prompt.md` | Оркестратор. Послідовно запускає `/git-init`, `/create-project`, `/create-build`, `/create-actions`, `/check`. |
| `/git-init` | `.github/prompts/git-init.prompt.md` | Перевіряє наявність `.git/`, виконує `git init` та формує `.gitignore`. |
| `/create-project` | `.github/prompts/create-project.prompt.md` | Створює каталоги `src/`, `tests/`, `docs/`, вихідний файл `src/main.cpp` та `src/addition.hpp`. |
| `/create-build` | `.github/prompts/create-build.prompt.md` | Генерує кореневий `CMakeLists.txt` та файл модульного тесту `tests/test_main.cpp`. |
| `/create-actions` | `.github/prompts/create-actions.prompt.md` | Генерує робочий процес `.github/workflows/ci.yml`, а також скрипти `ci.bat` та `ci.sh`. |
| `/check` | `.github/prompts/check.prompt.md` | Виконує перевірку файлової структури, компіляцію у Release, запуск CTest та аналіз індексу Git. |

### Приклад комплексного запуску
Для повного автоматичного розгортання та перевірки проєкту достатньо відправити команду:
```text
/init
```

---

## 2. Локальне складання та тестування

### Швидкий запуск через системні скрипти
- Платформа Windows:
  ```cmd
  ci.bat
  ```
- Платформи Linux / macOS:
  ```bash
  chmod +x ci.sh
  ./ci.sh
  ```

### Стандартні команди CMake та CTest
1. **Конфігурація системи складання:**
   ```bash
   cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
   ```
2. **Складання таргетів hello та project_tests:**
   ```bash
   cmake --build build --config Release
   ```
3. **Запуск застосунку:**
   - Windows:
     ```powershell
     .\build\Release\hello.exe
     ```
   - Linux / macOS:
     ```bash
     ./build/hello
     ```
4. **Запуск модульних тестів:**
   ```bash
   ctest --test-dir build -C Release --output-on-failure
   ```
