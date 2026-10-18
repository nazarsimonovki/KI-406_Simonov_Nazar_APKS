# Інструкція з запуску AI-агента та локальної збірки проєкту

## 1. Запуск AI-агента

Для ініціалізації та повного розгортання проєкту виконайте наступні кроки в середовищі VS Code:

1. Відкрийте палітру Copilot Chat.
2. Викликайте оркестратор `init` за допомогою промпту:
   ```text
   @workspace Прочитай agents/build-engineer.agent.md та виконай команду з prompts/init.prompt.md

Для виклику окремих команд використовуйте відповідні промпти з каталогу prompts/:

    prompts/git-init.prompt.md — перевірка та налаштування Git/gitignore.

    prompts/create-project.prompt.md — генерація вихідного коду C++.

    prompts/create-build.prompt.md — генерація CMakeLists.txt та тестів.

    prompts/create-actions.prompt.md — генерація CI workflow.

    prompts/check.prompt.md — перевірка стану проєкту.

## 2. Локальне складання та тестування проєкту
Для ручної локальної перевірки збірки та виконання тестів використайте наступні команди в терміналі:

Конфігурація CMake
```bash
cmake -S . -B build
```
Складання проєкту (генерація виконуваного файлу hello)
```bash
cmake --build build
```
Запуск юніт-тестів (BasicAddition)
```bash
cd build
ctest --output-on-failure
cd ..
```

