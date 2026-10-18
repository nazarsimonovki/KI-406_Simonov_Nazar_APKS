# Журнал взаємодії з AI-агентом (AI Interaction Log)

Цей документ фіксує послідовність взаємодії з AI-агентом (GitHub Copilot / Antigravity AI) у межах виконання Лабораторної роботи №1 («Проєктування AI-агента для автоматизації розробки та CI/CD», Варіант 1: C++, CMake, GoogleTest/CTest).

---

## 1. Початковий запит та ініціалізація структури (Крок `git-init` та `project-scaffold`)

### Запит (Prompt):
```text
@workspace Створи структуру проєкту для розробки AI-агента згідно з вимогами лабораторної роботи №1.
Налаштуй .gitignore для C++ та CMake (виключення папки build/, бінарників, кешів, секретів).
Ініціалізуй базовий C++ проєкт Hello World у src/main.cpp.
```

### Відповідь AI:
- Створено файл `.gitignore` з правилами для `build/`, `.cache/`, `*.exe`, `*.o`, `*.obj`.
- Створено каталог `src/` з файлом `main.cpp` та `src/addition.hpp` для майбутніх тестів.
- Створено каркас документації у `docs/`.

---

## 2. Налаштування системи складання та тестів (Крок `create-build` / `build-and-test`)

### Запит (Prompt):
```text
@workspace Створи конфігурацію CMake (CMakeLists.txt) та підключи GoogleTest через FetchContent.
Додай функцію add(int, int) у src/addition.hpp та юніт-тест BasicAddition у tests/test_main.cpp.
Налаштуй інтеграцію з CTest.
```

### Відповідь AI:
- Створено `CMakeLists.txt` із підтримкою C++17, включенням `CTest`, завантаженням `googletest` релізу v1.14.0 через `FetchContent` та налаштуванням `gtest_discover_tests`.
- Створено юніт-тест `TEST(BasicAddition, AddsTwoIntegers)` у `tests/test_main.cpp`.

---

## 3. Виявлена помилка №1: Невідповідність назви виконуваного файлу вимогам методички

### Опис проблеми:
У початковій конфігурації `CMakeLists.txt` було згенеровано таргет:
```cmake
add_executable(hello_world src/main.cpp)
```
Це призвело до генерації файлу `hello_world.exe` замість `hello` (або `hello.exe`), як чітко вимагається в методичних вказівках (Етап 4, п. 5: *"Перевірити, що створюється виконуваний файл hello, тест BasicAddition проходить..."*).

### Запит на виправлення (Prompt):
```text
У CMakeLists.txt назва таргету виконуваного файлу має бути саме hello, а не hello_world, щоб виконуваний файл мав назву hello (hello.exe). Виправ CMakeLists.txt та онови відповідні інструкції.
```

### Результат виправлення:
Таргет замінено на:
```cmake
add_executable(hello src/main.cpp)
```
Після повторного складання генерується файл `build/Release/hello.exe`, а тести `project_tests.exe` успішно проходять.

---

## 4. Виявлена помилка №2: Потрапляння файлу бази налагодження `tests/vc140.pdb` у Git

### Опис проблеми:
Під час компіляції під Windows компілятор MSVC згенерував налагоджувальний файл `tests/vc140.pdb`, який не був зазначений у правилах `.gitignore` і помилково потрапив під контроль версій Git. Методичка суворо забороняє наявність будь-яких проміжних файлів складання в репозиторії.

### Запит на виправлення (Prompt):
```text
Видали tests/vc140.pdb з індексу Git і додай розширення *.pdb, *.idb, *.ilk та папку .vs/ до .gitignore, щоб проміжні файли компілятора більше ніколи не потрапляли в коміти.
```

### Результат виправлення:
- Виконано `git rm --cached tests/vc140.pdb`.
- У `.gitignore` додано шаблони `*.pdb`, `*.idb`, `*.ilk`, `.vs/`.
- Перевірка через `git status` підтвердила чистоту репозиторію.

---

## 5. Налаштування CI/CD та завантаження артефактів (Крок `create-actions` / `github-actions`)

### Запит (Prompt):
```text
@workspace Налаштуй .github/workflows/ci.yml з матрицею Windows, Linux та macOS.
Workflow має запускатися на подіях push та pull_request для гілок develop, master та feature/**.
Всі кроки конфігурації та складання мають бути винесені в зовнішні скрипти ci.bat та ci.sh.
Додай крок upload-artifact для збереження зібраного виконуваного файлу hello як артефакту складання.
```

### Відповідь AI:
- Створено скрипти `ci.bat` (для Windows) та `ci.sh` (для Linux/macOS) із прапорцем `-DCMAKE_BUILD_TYPE=Release` та `--output-on-failure`.
- Оновлено workflow `.github/workflows/ci.yml` із кроком `actions/upload-artifact@v4`.

---

## 6. Результати локальної перевірки (Command: `check`)

Команда `check` виконала перевірку структури та складання:
- **Структура каталогів:** відповідає рекомендованій у розділі 5 методички (`.github/agents/`, `.github/prompts/`, `.github/skills/`, `.github/workflows/`, `src/`, `tests/`, `docs/`).
- **Локальна збірка:** успішна через `ci.bat`.
  ```text
  hello.vcxproj -> N:\LABS\APKS\LAB1\build\Release\hello.exe
  project_tests.vcxproj -> N:\LABS\APKS\LAB1\build\Release\project_tests.exe
  1/1 Test #1: BasicAddition.AddsTwoIntegers ....   Passed    0.01 sec
  100% tests passed, 0 tests failed out of 1
  ```
- **Стан Git:** відсутні build-каталоги, відсутні незаігноровані артефакти або секрети.
