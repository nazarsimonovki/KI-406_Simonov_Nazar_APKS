---
description: "Генерація GitHub Actions CI/CD workflow та скриптів автоматизації ci.bat і ci.sh"
---

# @create-actions.prompt.md

## Передумови
- Успішне виконання інструкції `@create-build.prompt.md`.
- Локальна збірка та тести проходять без помилок.
- Наявність навички `@github-actions` (`.github/skills/github-actions/SKILL.md`).

## Цільові файли та каталоги
- Каталог `.github/workflows/`
- Файл `.github/workflows/ci.yml`
- Файл `ci.bat`
- Файл `ci.sh`

## Дії
1. Створити у корені проєкту скрипт автоматизації для Windows `ci.bat`:
   ```cmd
   @echo off
   setlocal

   cmake -S . -B build
   if errorlevel 1 exit /b %errorlevel%

   cmake --build build --config Release
   if errorlevel 1 exit /b %errorlevel%

   ctest --test-dir build -C Release --output-on-failure
   exit /b %errorlevel%
   ```
2. Створити у корені проєкту скрипт автоматизації для Linux/macOS `ci.sh`:
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail

   cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
   cmake --build build --config Release
   ctest --test-dir build -C Release --output-on-failure
   ```
3. Створити файл workflow `.github/workflows/ci.yml`:
   - Назва: `CMake CI`
   - Тригери: `push` (гілки `develop`, `master`, `"feature/**"`) та `pull_request` (гілки `develop`, `master`)
   - Матриця операційних систем: `ubuntu-latest`, `windows-latest`, `macos-latest`
   - Кроки: `actions/checkout@v4`, виклик `ci.bat` на платформі Windows, виклик `bash ./ci.sh` на платформах Unix, збереження артефакту `hello` через `actions/upload-artifact@v4`.

## Очікуваний результат
- Файли `ci.bat` та `ci.sh` у корені проєкту.
- Файл `.github/workflows/ci.yml` із коректною CI-матрицею.

## Команди верифікації
- Перевірка наявності файлів `ci.bat`, `ci.sh`, `.github/workflows/ci.yml`.
- Синтаксична валідація YAML у `.github/workflows/ci.yml`.

## Обробка помилок
- У разі виникнення збою вивести: "Помилка @create-actions.prompt.md: не вдалося згенерувати скрипти складання або валідний файл .github/workflows/ci.yml."
