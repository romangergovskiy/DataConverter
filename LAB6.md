# Лаба 6 — CI для DataConverter

Лабы 6 и 7 **не связаны**. Это отдельный проект: конвертер CSV/JSON/YAML + GitHub Actions.

## Что сделано

- Репозиторий с исходниками DataConverter (C++17, CMake, GoogleTest).
- Workflow `.github/workflows/ci.yml`:
  - триггеры `push` и `pull_request`;
  - матрица `ubuntu-latest` / `macos-latest` / `windows-latest`;
  - `fail-fast: false`;
  - кэш `build/_deps` (yaml-cpp и GoogleTest через FetchContent);
  - артефакты: отчёт тестов `test-results.xml` и бинарник.

## Локальный запуск

```bash
cd ~/Downloads/antonlabs/lab6/DataConverter
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
./build/dataconv examples/people.csv json
```

Нужны CMake 3.20+, компилятор C++17 и интернет при первой сборке (качаются yaml-cpp и GoogleTest).

## GitHub

После `git push` откройте вкладку **Actions**. Должны быть три зелёные джобы. Скачайте артефакт `test-report-ubuntu-latest`.

## Часть 4 — fail-fast (для рассказа, тесты уже целые)

1. `fail-fast: false` + сломанный тест → падают все три ОС, но **все три джобы всё равно запускаются**.
2. `fail-fast: true` + тот же сломанный тест → как только упала первая ОС, остальные GitHub **может отменить**.
3. Тест вернули, `fail-fast` оставили `false`.

## Чем кэш отличается от артефакта

- **Кэш** — для ускорения следующих сборок (зависимости). Внутренний, по ключу ОС + `CMakeLists.txt`.
- **Артефакт** — результат прогона, который можно скачать (отчёт, exe). Лежит у конкретного workflow run.

Кэш по ОС раздельный: Linux-библиотеки бесполезны на Windows.

Почему Ubuntu ≠ Windows: пути, генератор CMake, CRT, расширения `.exe`, разные компиляторы.

## Вопросы на защите (коротко)

- CI — автоматически собирать и тестировать каждый push, чтобы ловить поломки сразу.
- Workflow — YAML в `.github/workflows`.
- `push` — в ветку; `pull_request` — проверка чужого PR до мержа.
- Job — задание (одна ОС), step — шаг внутри.
- `runs-on` — какая машина (ubuntu/mac/windows).
- `matrix` — одно и то же job × набор ОС.
- Задания матрицы изолированы: свой runner, свои зависимости.
