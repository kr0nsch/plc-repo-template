# Как работать с шаблоном

Пошаговое руководство для своего ПЛК-проекта на этом шаблоне: создать репозиторий, убрать пример,
работать каждый день (в текстовом редакторе или в IDE CODESYS), добавить свой POU с тестом, проверить
в CI и выпустить релиз; перенести на шаблон существующий проект. Коротко — в [README](../README.md#как-использовать-в-своём-проекте); установка
CODESYS и coUnit — в [MANUAL_STEPS.md](MANUAL_STEPS.md); все опции и переменные окружения — в
[tools/README.md](../tools/README.md).

Команды — из корня репозитория. Команды `cds.py` работают одинаково в cmd, PowerShell и Git Bash; в
примерах ниже пути записаны через `\` (cmd, PowerShell), в Git Bash пишите `/`.

## Содержание

1. [Создать репозиторий из шаблона](#1-создать-репозиторий-из-шаблона)
2. [Переименовать под свой проект](#2-переименовать-под-свой-проект)
3. [Удалить пример](#3-удалить-пример)
4. [Ежедневная работа](#4-ежедневная-работа)
5. [Добавить POU и тест](#5-добавить-pou-и-тест)
6. [Задачи, устройство и библиотеки: config.json](#6-задачи-устройство-и-библиотеки-configjson)
7. [Команды cds.py](#7-команды-cdspy)
8. [CI](#8-ci)
9. [Релиз](#9-релиз)
10. [Перенос существующего проекта](#10-перенос-существующего-проекта)

## 1. Создать репозиторий из шаблона

1. На GitHub: **Use this template → Create a new repository**.
2. Клонировать. Имя каталога — имя проекта CODESYS по умолчанию: `build\<имя>.project` и
   `build\<имя>-tests.project` (другое имя — переменная `CDS_PROJECT_NAME`).

   ```bash
   git clone https://github.com/<владелец>/<имя>.git
   cd <имя>
   ```

3. Окружение Python и проверки без CODESYS — как в [README, «Быстрый старт»](../README.md#быстрый-старт).
   На исходниках шаблона все проверки проходят: CI нового репозитория — тот же
   `.github/workflows/ci.yml`, он запускается с первого push.

Уже есть проект CODESYS, который нужно перевести на шаблон, — после раздела 2 переходите к
[разделу 10](#10-перенос-существующего-проекта): пример заранее удалять не нужно.

## 2. Переименовать под свой проект

Все места с автором и именем шаблона:

```bash
git grep -n -e kr0nsch -e plc-repo-template -- . ":!tools/tests"
```

- [ ] `LICENSE` — правообладатель.
- [ ] `README.md` — переписать под проект: описание, адреса в бейджах, подвал портфолио убрать.
- [ ] `CONTRIBUTING.md` — раздел «Контекст» описывает портфолио автора шаблона; правила кода
      оставьте или измените под команду.
- [ ] Заголовки POU в `src/` и `tests/` — строка `Автор:`.
- [ ] `tools/codesys/config.json` → `library`: `title`, `company`, `default_namespace` (сейчас `TPL`),
      `author`, `description`. Компанию и название задайте до первого выпуска библиотеки и потом не
      меняйте: по ним библиотеку находят проекты-потребители ([MANUAL_STEPS.md, 7.1](MANUAL_STEPS.md#71-выпуск-релиза)).
- [ ] `pyproject.toml` → `[project] name` (сейчас `plc-repo-template-tools`).
- [ ] `docs/media/` — баннеры (`banner-*.svg`, `banner-*.png`) и `social-preview.png` содержат имя
      шаблона, GIF и снимок IDE показывают шаблон: замените своими или удалите вместе со ссылками.
- [ ] `CHANGELOG.md` — начать свою историю.
- [ ] Журналы в `docs/` с записями шаблона: `ASSUMPTIONS.md` (решения шаблона — оставить как справку
      или очистить), `STATIC_ANALYSIS.md` (ссылки на прогоны CI шаблона), `BUILD_ERRORS.md` (запись
      2026-10-19), `TEST_REPORT.md` (раздел прогона перезапишет `cds.py report`), `MANUAL_STEPS.md`
      (имя шаблона в примерах).

`tools/tests/` не трогайте: эталоны тестов инструментов от имени проекта не зависят.

## 3. Удалить пример

Пример — насос подкачки `FB_Example` с командами и состоянием для HMI ([ARCHITECTURE.md](ARCHITECTURE.md#пример-в-src)).
Шаги ниже проверены на копии шаблона: после них `st_lint`, iec_checker, RuSTy и `pytest` проходят без
замечаний. Если пропустить шаг 5, CI упадёт на `pytest`.

1. Удалить объекты примера и его тесты:

   ```bash
   git rm -r src/POUs/FB_Example src/DUTs/ST_ExampleCmd.dut.st src/DUTs/ST_ExampleSts.dut.st tests/POUs/FB_Example_Test
   ```

2. `src/GVLs/GVL_Const.gvl.st` — удалить константы `EXAMPLE_*`. Пустой блок
   `VAR_GLOBAL CONSTANT` / `END_VAR` допустим.
3. `src/POUs/PRG_Main/` — убрать из объявления и тела переменные примера (`fbPump`, `itfResettable`,
   `stCmd`, `stSts`, `rLevel_pct`, `xLevelValid`, `xPumpOn`) и код, который их использует. Пустое тело —
   файл с одним комментарием.
4. `tests/POUs/PRG_Tests/` — удалить объявление `fbExampleTest : FB_Example_Test;` и
   `AND fbExampleTest.AreAllTestsFinished()` в теле.
5. `tools/check/iec_checker_allow.toml` — удалить обоснования, которые относились к удалённому коду:
   `src/POUs/FB_Example/*`, `src/POUs/FB_Example/FB_Example.impl.st`,
   `tests/POUs/FB_Example_Test/*.method.decl.st`, а для `PRG_Main` — `PRG_Main.impl.st` (PLCOPEN-L17)
   и `PRG_Main.decl.st` (`STCMD|STSTS`), если этого кода больше нет. Какие записи не используются,
   показывает сам запуск:

   ```bash
   python tools/check/run_iec_checker.py --tests
   ```

   ```text
   run_iec_checker: предупреждение: обоснование PLCOPEN-CP9 / src/POUs/FB_Example/* не использовано
   ...
   ```

   Неиспользованное обоснование роняет `pytest`
   (`tools/tests/test_run_iec_checker.py::test_integration_repository_sources`) — обязательный шаг CI.
6. Документы с примером: `docs/ARCHITECTURE.md` (раздел «Пример в src/»), `docs/STATIC_ANALYSIS.md`
   (находки `FB_Example`, `PRG_Main`, `FB_Example_Test`), `docs/TEST_REPORT.md` («Что проверяют
   тесты»), `docs/PMI_template.md` (две строки-примера), `config.json` → `library.description`.
7. Проверить:

   ```bash
   python tools/check/st_lint.py --strict
   python tools/check/run_iec_checker.py --tests   # без предупреждений «не использовано»
   python tools/check/run_rusty.py                 # находит обращения к удалённым объектам
   python -m pytest
   ```

   и, если есть CODESYS, `python tools\codesys\cds.py check --tests`. Без RuSTy и CODESYS забытое
   обращение к удалённому объекту (например, `fbPump` в `PRG_Main`) не найдёт ни `st_lint`, ни
   iec_checker.

Остальные объекты — `E_DeviceMode`, `I_Resettable`, `F_Clamp` с набором `FB_F_Clamp_Test` — от
`FB_Example` не зависят: оставьте как образец или удалите тем же порядком (после любого удаления —
`run_iec_checker.py --tests` и чистка неиспользованных обоснований).

## 4. Ежедневная работа

Источник истины — файлы в `src/` и `tests/`. Проекты в `build\` — производные: их собирает
`cds.py import`, в git они не попадают (`.gitignore`). Править можно двумя способами.

### Маршрут А: правка текста

Для ST-кода в любом редакторе (VS Code, Notepad++ и т. п.):

1. Изменить файлы `src/**/*.st` и `tests/**/*.st` по раскладке [LAYOUT.md](LAYOUT.md#типы-файлов).
2. Проверить без CODESYS:

   ```bash
   python tools/check/st_lint.py --strict
   ```

3. С CODESYS — сборка, полная проверка компиляции и тесты:

   ```bat
   python tools\codesys\cds.py build
   python tools\codesys\cds.py check --tests
   python tools\codesys\cds.py test
   ```

4. `git add`, `git commit`, `git push` — CI проверит то же, что шаг 2, и больше (раздел 8).

### Маршрут Б: правка в IDE CODESYS

Когда удобнее редактор CODESYS: онлайн-режим, отладка, автодополнение.

1. Собрать проект из текстов:

   ```bat
   python tools\codesys\cds.py import
   ```

   С тестами — `import --tests` (проект `build\<имя>-tests.project`: `src/` и `tests/` в папке `Tests`,
   задача `TestTask` → `PRG_Tests`, библиотека coUnit).
2. Открыть в IDE: «Файл → Открыть проект» → `build\<имя>.project`. Из консоли и вход в рантайм —
   [MANUAL_STEPS.md, раздел 6](MANUAL_STEPS.md#6-работа-в-ide-и-экспорт-перед-коммитом).
3. Править, сохранить проект и **закрыть IDE**. Пока проект открыт, команды, которые открывают этот же
   файл, отказываются с кодом выхода 3 («проект … открыт в другом экземпляре CODESYS»): для
   `build\<имя>.project` — `import`, `build`, `export`; для `build\<имя>-tests.project` —
   `import --tests`, `build --tests`, `export --tests` и `test`.
4. Выгрузить правки в тексты и посмотреть дифф:

   ```bat
   python tools\codesys\cds.py export
   git diff --stat
   ```

   Для проекта с тестами — `export --tests` (пишет и `src/`, и `tests/`).
5. Проверить (`st_lint --strict`, `cds.py build`, `check --tests`, `test`) и закоммитить.

Задачи и ссылки на библиотеки в IDE не правьте: их создаёт `import` по `tools/codesys/config.json`
(раздел 6), лишние задачи удаляются. Визуализации, дерево устройств, конфигураторы обмена,
графические языки и `ACTION` в раскладку не входят и при экспорте не сохраняются
([MANUAL_STEPS.md, раздел 6](MANUAL_STEPS.md#6-работа-в-ide-и-экспорт-перед-коммитом)).

### Защита от потери правок

- `import` не перезапишет проект, сохранённый в IDE после последнего `import`/`export`: сначала
  `export`, или `--force`, если правки в IDE не нужны.
- `export` не перезапишет `src/` (`tests/`), если они изменились после последнего `import`/`export`
  этого проекта: экспорт удаляет файлы, которых нет в проекте. Сравнить — выгрузкой в отдельный
  каталог `cds.py export --to build\cmp`; перезаписать — `--force`.

Отпечатки проекта и исходников хранятся в `build/codesys/state.json`
([tools/README.md](../tools/README.md#блокировки-и-защита-от-потери-правок)).

## 5. Добавить POU и тест

Пример — функция `F_Clamp` из шаблона и её набор тестов. Файлы:

```text
src/POUs/F_Clamp/
  F_Clamp.decl.st                      заголовок и объявление: FUNCTION F_Clamp : REAL, VAR_INPUT, VAR
  F_Clamp.impl.st                      тело
tests/POUs/FB_F_Clamp_Test/
  FB_F_Clamp_Test.decl.st              FUNCTION_BLOCK FB_F_Clamp_Test EXTENDS coUnit.FB_TestSuite
  FB_F_Clamp_Test.impl.st              вызовы методов-тестов
  BelowLowReturnsLow.method.decl.st    METHOD PRIVATE BelowLowReturnsLow и его переменные
  BelowLowReturnsLow.method.impl.st    тело теста
  ...                                  ещё 6 тестов — по паре файлов на метод
tests/POUs/PRG_Tests/                  регистрация набора (шаг 3)
```

**1. POU.** Каждый POU начинается с комментария-заголовка: назначение, автор, версия и путь к
документу в `docs/` — без него `st_lint` выдаёт ошибку STL001. `src/POUs/F_Clamp/F_Clamp.decl.st`:

```st
// Назначение: ограничение значения отрезком между двумя границами.
//             Если rLow > rHigh, границы меняются местами: результат всегда лежит
//             между двумя заданными границами и не зависит от порядка аргументов.
//             NaN на входе не обрабатывается — достоверность проверяется до вызова.
// Автор:      kr0nsch
// Версия:     0.1.0
// Документ:   docs/ASSUMPTIONS.md, раздел «Семантика F_Clamp»
FUNCTION F_Clamp : REAL
VAR_INPUT
    rValue : REAL;      // ограничиваемое значение
    rLow : REAL;        // нижняя граница
    rHigh : REAL;       // верхняя граница
END_VAR
VAR
    rMin : REAL;        // нижняя граница после упорядочивания
    rMax : REAL;        // верхняя граница после упорядочивания
END_VAR
```

Имена — по [CONTRIBUTING.md](../CONTRIBUTING.md#именование-st): префиксы POU (`PRG_`, `FB_`, `F_`) и
переменных по типу (`r` — REAL, `x` — BOOL, `fb` — экземпляр ФБ), константы — в `GVL_Const`. Методы и
свойства ФБ — отдельные файлы (`<Метод>.method.decl.st`, `<Свойство>.prop.decl.st`,
`<Свойство>.get.impl.st`), см. [LAYOUT.md](LAYOUT.md#типы-файлов).

**2. Набор тестов coUnit.** Набор — ФБ `EXTENDS coUnit.FB_TestSuite`, каждый тест — его метод; тело
набора вызывает методы каждый цикл. Тест `tests/POUs/FB_F_Clamp_Test/BelowLowReturnsLow.method.impl.st`:

```st
coUnit.TEST('BelowLowReturnsLow');

rResult := F_Clamp(rValue := R_BELOW, rLow := R_LOW, rHigh := R_HIGH);
AssertEquals_REAL(Expected := R_LOW,
                  Actual := rResult,
                  Delta := R_DELTA,
                  Message := 'Value below range must be clamped to rLow');

coUnit.TEST_FINISHED();
```

`coUnit.TEST` и `coUnit.TEST_FINISHED` — с пространством имён, методы проверки (`AssertEquals_REAL`,
`AssertTrue` и др.) — без него: это методы базового ФБ. Строки `STRING` — только ASCII (правило
STL010). Константы набора (`R_LOW`, `R_BELOW`, ...) — в `VAR CONSTANT` объявления набора.

**3. Регистрация в `PRG_Tests`.** Набор регистрируется в coUnit объявлением экземпляра в `PRG_Tests`;
`coUnit.RUN()` выполняет все зарегистрированные наборы. `cds.py test` ждёт признака раннера coUnit
`AllTestSuitesFinished` (`config.json` → `tests.runner`), а не флага программы. Флаг `xAllFinished` —
сводный признак окончания наборов внутри `PRG_Tests`; новый набор добавляют в него так же, как в
шаблоне. `tests/POUs/PRG_Tests/PRG_Tests.decl.st`, сокращено:

```st
VAR
    fbF_ClampTest : FB_F_Clamp_Test;    // тесты F_Clamp
    fbExampleTest : FB_Example_Test;    // тесты FB_Example
    xAllFinished : BOOL;
END_VAR
```

`tests/POUs/PRG_Tests/PRG_Tests.impl.st`:

```st
coUnit.RUN();
xAllFinished := fbF_ClampTest.AreAllTestsFinished() AND fbExampleTest.AreAllTestsFinished();
```

**4. Задача для новой программы.** Новый `PROGRAM` получает задачу через `tools/codesys/config.json`
→ `modes.normal.tasks` (раздел 6), а не в IDE.

**5. Проверить:**

```bash
python tools/check/st_lint.py --strict
python tools/check/run_iec_checker.py --tests
```

```bat
python tools\codesys\cds.py check --tests
python tools\codesys\cds.py test
```

`cds.py test` выведет итог по наборам, например
`[cds] test: OK (наборов 2, тестов 18, пройдено 18, упало 0, пропущено 0, 9.1 с)`; упавшие тесты — с
сообщением coUnit. Итоги — в `build/test-results.json` и `build/junit.xml`, в отчёт —
`python tools\codesys\cds.py report` (раздел «Последний прогон в CODESYS» в
[TEST_REPORT.md](TEST_REPORT.md)). Новые находки iec_checker исправьте в коде или обоснуйте в
`tools/check/iec_checker_allow.toml` и [STATIC_ANALYSIS.md](STATIC_ANALYSIS.md).

## 6. Задачи, устройство и библиотеки: config.json

`tools/codesys/config.json` — единственное место настроек проекта CODESYS; между репозиториями на
шаблоне отличается только он.

**Задачи** — по режимам: `normal` для рабочего проекта, `test` для проекта с тестами. Сейчас:

```json
"modes": {
  "normal": {
    "tasks": [{"name": "MainTask", "interval_ms": 20, "pous": ["PRG_Main"]}],
    "libraries": []
  },
  "test": {
    "tasks": [{"name": "TestTask", "interval_ms": 10, "pous": ["PRG_Tests"]}],
    "libraries": ["counit"]
  }
}
```

**Устройство** — раздел `device`. По умолчанию CODESYS Control Win V3, 32 бит (`type` 4096, `id`
`0000 0001`, версия 3.5.17.30) — на нём проверены сборка и тесты. Другие варианты перечислены в
`device._alternatives`; для ОВЕН ПЛК210 тип, идентификатор и версию берут из таргет-файла ОВЕН после
установки пакета ([MANUAL_STEPS.md, раздел 1](MANUAL_STEPS.md#1-установка-codesys-351730)) — на ПЛК210
шаблон не проверялся. На одну сборку устройство можно задать переменными `CDS_DEVICE_TYPE`,
`CDS_DEVICE_ID`, `CDS_DEVICE_VERSION`.

**Библиотеки.** Standard (TON, R_TRIG, CTU, CONCAT) подключена по умолчанию плейсхолдером `#Standard`.
Своя или чужая библиотека описывается в `libraries` и подключается из `modes.<режим>.libraries`;
библиотека из соседнего репозитория (например, plc-process-lib) собирается автоматически —
[MANUAL_STEPS.md, 7.3](MANUAL_STEPS.md#73-проект-который-использует-библиотеку-портфолио).

Все ключи — в [tools/README.md](../tools/README.md#codesyscdspy--codesys-из-консоли).

### Другая версия или дистрибутив CODESYS

Проверена только 32-битная сборка ОВЕН CODESYS V3.5 SP17 Patch 3 (3.5.17.30). С другими версиями V3.5
и со стандартным дистрибутивом CODESYS Group (в том числе 64-битным) шаблон **не запускался**. Значения,
привязанные к версии, по коду `cds.py` задаются так:

| Что | Переменная или ключ `config.json` | По умолчанию |
|---|---|---|
| `CODESYS.exe` | `CODESYS_EXE` (`codesys.exe`) | `CODESYS 3.5.17.30\CODESYS\Common\CODESYS.exe` в `%ProgramFiles(x86)%`, затем в `%ProgramFiles%` |
| профиль | `CODESYS_PROFILE` (`codesys.profile`) — по файлу в каталоге `Profiles` установки ([MANUAL_STEPS.md, раздел 1](MANUAL_STEPS.md#1-установка-codesys-351730)) | `CODESYS V3.5 SP17 Patch 3` |
| версия устройства | `CDS_DEVICE_VERSION` (`device.version`) — по репозиторию устройств этой установки | `3.5.17.30` |
| 64-битный Control Win V3 | `CDS_DEVICE_ID` = `0000 0004` (`device.id`, варианты — `device._alternatives`) | `0000 0001` (32 бит) |
| рантайм для `cds.py test` | `CODESYS_RUNTIME_EXE`, `CODESYS_RUNTIME_CFG_TEMPLATE` | `CODESYS 3.5.17.30\GatewayPLC\CODESYSControlService.exe`; `%ProgramData%\CODESYS\CODESYSControlWinV3\*\CODESYSControl.cfg` |
| версия в отчёте тестов | `codesys.version` — только подпись в `build/test-results.json` и `docs/TEST_REPORT.md` | `3.5.17.30` |

Переменные действуют на одну консоль или, через `setx`, на пользователя; постоянное значение для
репозитория — правка `config.json`. В репозитории библиотек этой установки должна быть coUnit 1.2.0.0
([MANUAL_STEPS.md, раздел 2](MANUAL_STEPS.md#2-counit-cfunit-1200)); версию Standard выбирает профиль.
Если устройство уже собранного проекта в `build\` не совпадает с `config.json`, `import` отказывается и
предлагает пересоздать проект: `cds.py import --clean`.

### Целевые ПЛК: ОВЕН ПЛК210, СПК210 и REGUL R500

Устройство проекта меняется переменными окружения — `config.json` и исходники остаются прежними. Проект
удобно собирать в отдельный файл (`CDS_PROJECT`, `CDS_TEST_PROJECT`), чтобы не пересоздавать проект Control Win
(`import` отказывается работать с проектом другого устройства).

| Целевой ПЛК | Среда | `CDS_DEVICE_TYPE` / `CDS_DEVICE_ID` / `CDS_DEVICE_VERSION` | Ещё |
|---|---|---|---|
| ОВЕН ПЛК210-11…15, -4G | CODESYS 3.5.17.30 + пакет OwenTargets 3.5.17.36 | `4096` / `1628 0083` / `3.5.17.35` | `CDS_DEVICE_TITLE="ОВЕН ПЛК210"` (подпись в отчётах) |
| ОВЕН СПК210 | то же | `4096` / `1628 0073` / `3.5.17.34` | не собиралось |
| REGUL R500 61 | Astra.IDE 1.7.2.2 | `4096` / `109B 0008` / `11.7.2.60` | `CODESYS_EXE` = `Astra.IDE.exe`, `CODESYS_PROFILE=Astra.IDE_V1.7.2.2`, свои `CDS_LIBRARY_REGISTRY` и `CDS_LIBRARY_LOCK` |

Пакет таргетов ОВЕН — с `https://ftp.owen.ru/CoDeSys3/03_Targets/` (версию под прошивку — по матрице
`https://ftp.owen.ru/CoDeSys3/96_CdsVersionSelector/cds_version_selector.json`), установка — «Инструменты →
Менеджер пакетов» или `PackageManagerCLI.exe --profile="CODESYS V3.5 SP17 Patch 3" --install="<файл>.package"`
(права администратора). У Astra.IDE свой репозиторий библиотек (`C:\ProgramData\Astra.IDE_64_1.7.2.2\Managed
Libraries`): учёт установок `cds.py` для неё — отдельными файлами, иначе он смешается с учётом CODESYS.

```bat
:: ОВЕН ПЛК210 (cmd)
set CDS_DEVICE_ID=1628 0083
set CDS_DEVICE_VERSION=3.5.17.35
set CDS_DEVICE_TITLE=ОВЕН ПЛК210
set CDS_PROJECT=build\plc210\plc-repo-template.project
set CDS_TEST_PROJECT=build\plc210\plc-repo-template-tests.project
python tools\codesys\cds.py build
python tools\codesys\cds.py check --tests

:: REGUL R500 61 в Astra.IDE (новая консоль)
set CODESYS_EXE=C:\Program Files\AstraRegul\Astra.IDE_64_1.7.2.2\Astra.IDE\Common\Astra.IDE.exe
set CODESYS_PROFILE=Astra.IDE_V1.7.2.2
set CDS_DEVICE_ID=109B 0008
set CDS_DEVICE_VERSION=11.7.2.60
set CDS_DEVICE_TITLE=REGUL R500 61
set CDS_LIBRARY_REGISTRY=%LOCALAPPDATA%\plc-tools\astra-libraries.json
set CDS_LIBRARY_LOCK=%LOCALAPPDATA%\plc-tools\astra-library-repo.lock
set CDS_PROJECT=build\r500\plc-repo-template.project
python tools\codesys\cds.py build
python tools\codesys\cds.py check
```

Устройство ПЛК210 само создаёт задачу `OwenCloudTask` (облако ОВЕН): импорт её не удаляет — ключ
`device.keep_tasks` в `config.json`; без него сборка даёт предупреждение C0373. Тесты coUnit выполняются только в
CODESYS Control Win: на ПЛК210 тестовый проект лишь собирается (`check --tests`), в Astra.IDE coUnit не
установлена.

Проверено 20.10.2026 (только компиляция, в ПЛК не загружалось):

| Устройство | `cds.py build` | `cds.py check` | Тестовый проект |
|---|---|---|---|
| ОВЕН ПЛК210 (CODESYS 3.5.17.30, OwenTargets 3.5.17.36) | 0 ошибок, 0 предупреждений | 8 объектов, 0 ошибок | `check --tests` — 11 объектов, `build --tests` — 0 ошибок, 0 предупреждений |
| REGUL R500 61 (Astra.IDE 1.7.2.2) | 0 ошибок, 0 предупреждений | 8 объектов, 0 ошибок | — (coUnit нет в Astra.IDE) |

## 7. Команды cds.py

Нужны Windows, CODESYS 3.5.17.30 и Python 3.11+ (сторонние пакеты не нужны). `cds.py` запускает
CODESYS без окна (`--noUI`) со скриптами CODESYS Scripting; тесты coUnit — в CODESYS Control Win V3,
запущенном пользовательским процессом, без прав администратора. Каждая команда длится 10–20 секунд и
пишет журналы в `build/`.

| Команда (`python tools\codesys\cds.py ...`) | Что делает |
|---|---|
| `import [--tests]` | `src/` → `build\<имя>.project`; с `--tests` — `src/` и `tests/` → `build\<имя>-tests.project` |
| `build [--tests]` | импорт и полная сборка; ошибки — с файлом и строкой в `src/` (`build/build.log`) |
| `check [--tests]` | компиляция **всех** объектов `src/` (и `tests/`), в том числе не вызываемых из задач (`build/check.log`) |
| `test` | `src/` + `tests/` → сборка → прогон в рантайме → `build/test-results.json`, `build/junit.xml` |
| `roundtrip` | импорт в чистый проект → экспорт → побайтное сравнение с `src/` и `tests/` |
| `export [--tests] [--to DIR]` | после правок в IDE: проект → `src/` (и `tests/`) |
| `report` | итоги последнего `test` → `docs/TEST_REPORT.md` |
| `library [--install] [--compiled] [--released]` | `src/` → библиотека `build\<имя>.library`, установка в репозиторий библиотек |
| `deps [KEY...] [--check] [--rebuild]` | библиотеки-зависимости: проверка, установка, сборка в соседних репозиториях |
| `runtime status` | рантайм; также `start`, `stop`, `restart`, `init` |

`build` компилирует только объекты, достижимые из задач: ошибка в ФБ без экземпляра или в
неиспользуемом DUT даёт «0 ошибок». Такие ошибки находит `check`. Поэтому полная проверка перед
коммитом и релизом — четыре команды:

```bat
python tools\codesys\cds.py build
python tools\codesys\cds.py check --tests
python tools\codesys\cds.py test
python tools\codesys\cds.py roundtrip
```

Итоговые строки на шаблоне (CODESYS 3.5.17.30, [TEST_REPORT.md](TEST_REPORT.md); время у вас будет другим):

```text
[cds] build: OK (ошибок 0, предупреждений 0, 1.0 с)
[cds] check: все объекты src/ и tests/ скомпилированы без ошибок (11 объектов)
[cds] test: OK (наборов 2, тестов 18, пройдено 18, упало 0, пропущено 0, 9.1 с)
[cds] roundtrip: экспорт совпадает с src/ и tests/ побайтно (60 файлов)
[cds] roundtrip: повторный импорт после открытия проекта с диска — 0 изменений
```

Коды выхода: 0 — успех; 1 — проверка не прошла (ошибки раскладки, сборки, упавшие тесты, дифф
roundtrip, защита от потери правок); 2 — аргументы или `config.json`; 3 — окружение (нет CODESYS,
рантайма, файла; проект открыт в IDE); 4 — таймаут; 5 — блокировка занята. Что смотреть в выводе
каждой команды — [MANUAL_STEPS.md, раздел 5](MANUAL_STEPS.md#5-импорт-сборка-тесты); все опции —
`python tools\codesys\cds.py <команда> --help` и
[tools/README.md](../tools/README.md#codesyscdspy--codesys-из-консоли).

## 8. CI

Workflow `.github/workflows/ci.yml` запускается на каждый push, pull request и вручную
(`workflow_dispatch`). CODESYS в нём нет: GitHub-hosted раннеры — Linux, лицензия не нужна.

| Задание | Обязательно | Шаги | Артефакт |
|---|---|---|---|
| Python, сборка ST, линтер, iec_checker | да | `ruff check`, `ruff format --check`, `mypy`, `pytest`, `assemble.py --tests`, `st_lint.py --strict`, `run_iec_checker.py --tests` (iec_checker v0.4, бинарник закреплён по размеру и SHA-256) | `build-reports` |
| iec_checker nightly, Docker | нет (`continue-on-error`) | тот же статанализ образом `nightly`, закреплённым по дайджесту | `iec-checker-nightly-reports` |
| RuSTy plc --check | нет (`continue-on-error`) | `run_rusty.py` (RuSTy v1.0.6, файлы закреплены по SHA-256) | `rusty-reports` |

Итоги каждого задания — в сводке запуска (Summary), полные отчёты — в артефактах (хранятся 14 дней).
Шаги обязательного задания выполняются все, даже если один упал: сводка показывает полную картину.

Сборка и тесты в CODESYS в этот CI не входят. Перед слиянием и релизом их запускают на Windows с
CODESYS (раздел 7) или на своём Windows-раннере — переиспользуемые workflow для репозиториев на этом
шаблоне есть в [plc-ci-toolkit](https://github.com/kr0nsch/plc-ci-toolkit).

## 9. Релиз

`.project` и `.library` в git не хранятся — их прикладывают к релизу GitHub. Кратко:

1. Полная проверка (раздел 7): `build`, `check --tests` — 0 ошибок, `test` — все тесты пройдены,
   `roundtrip` — без диффа.
2. Версия — запись в `CHANGELOG.md`; для библиотеки — `library.version` в `config.json`.
3. Для репозитория-библиотеки: `python tools\codesys\cds.py library --compiled --released`.
4. Приложить к релизу `build\<имя>.project` (и библиотеки), поставить тег `vX.Y.Z`, обновить отчёт —
   `python tools\codesys\cds.py report`.

Подробно, с установкой библиотек и зависимостями между репозиториями, —
[MANUAL_STEPS.md, раздел 7](MANUAL_STEPS.md#7-релиз-и-библиотеки-project-library-зависимости).

## 10. Перенос существующего проекта

> **На реальном проекте не проверялось.** Порядок составлен по коду `cds.py`, `export_sources.py` и
> `import_sources.py`; команды `export`, `import --clean`, `build`, `check` и `roundtrip` выполнялись
> только на проектах, собранных из исходников самого шаблона. Работайте с копией проекта и сверяйте
> результат с оригиналом.

**Что переносится.** Объекты активного приложения (Application) на ST: DUT, GVL, интерфейсы с методами
и свойствами, POU (`PROGRAM`, `FUNCTION_BLOCK`, `FUNCTION`) с методами и свойствами. Папки проекта
становятся подкаталогами: `Application/Motors/FB_Motor` → `src/POUs/Motors/FB_Motor/`. Папка верхнего
уровня `Tests` считается тестами и выгружается в `tests/` только с `--tests`.

**Что не переносится:**

| Что | Что делает `export` | Как быть |
|---|---|---|
| действия (`ACTION`) | пропускает, предупреждает | переписать в методы ФБ |
| POU на графических языках (FBD, LD, CFC, SFC) | пропускает, предупреждает | переписать на ST или оставить в исходном проекте |
| методы на графических языках | выгружает только объявление, предупреждает | переписать тело на ST |
| визуализации, текстовые списки и другие объекты внутри приложения | пропускает, предупреждает | ручной шаг в IDE ([MANUAL_STEPS.md, раздел 6](MANUAL_STEPS.md#6-работа-в-ide-и-экспорт-перед-коммитом)) |
| дерево устройств: ПЛК, модули ввода-вывода, привязка каналов, конфигураторы обмена | не читает | устройство — `config.json` → `device` (раздел 6), остальное — ручной шаг |
| задачи и ссылки на библиотеки | не выгружает | описать в `config.json` (шаг 4) |
| Persistent Variables | выгружает в `*.pvl.st` | в SP17 скриптом не создаются — объект создают вручную ([MANUAL_STEPS.md, раздел 6](MANUAL_STEPS.md#6-работа-в-ide-и-экспорт-перед-коммитом)) |
| объекты вне активного приложения (пул POU уровня проекта, другие приложения) | не читает | в копии проекта перенести в приложение до экспорта |

Не запускайте `cds.py import` на оригинальном проекте (например, через `CDS_PROJECT`): импорт удаляет из
приложения POU, DUT, GVL и интерфейсы, которых нет в `src/`, — в том числе POU на графических языках,
которые экспорт пропустил.

**Порядок:**

1. Репозиторий из шаблона и переименование — разделы 1 и 2. Пример заранее удалять не нужно: экспорт
   перезапишет `src/`, а тесты шаблона убираются на шаге 5.
2. Закрыть проект в IDE и скопировать его в `build\` под именем проекта репозитория (`<имя>` — имя
   каталога репозитория, раздел 1). Оригинал не меняется; команды — для cmd:

   ```bat
   if not exist build mkdir build
   copy "D:\Projects\Line1\Line1.project" build\<имя>.project
   ```

3. Посмотреть, что выгрузится, не трогая `src/`, затем выгрузить в репозиторий:

   ```bat
   python tools\codesys\cds.py export --to build\migrate
   python tools\codesys\cds.py export --force
   ```

   Первая команда пишет `build\migrate\src\`. Пропущенные объекты перечислены в `build\export.log` и в
   поле `warnings` файла `build\export.json` (в консоль выводится только число файлов и объектов).
   Вторая пишет `src/` и удаляет из него файлы примера шаблона, которых нет в проекте; `--force`
   отключает защиту от потери правок — `src/` здесь перезаписывается намеренно.
4. Описать в `tools/codesys/config.json` то, что экспорт не выгружает:
   - задачи — в `modes.normal.tasks`: имя, интервал, программы. Импорт создаёт циклические задачи
     (ключ `priority` код импорта тоже понимает, в шаблоне он не проверялся); задачи, которых нет в
     `config.json`, следующий `import` удаляет:

     ```json
     "tasks": [
       {"name": "MainTask", "interval_ms": 10, "pous": ["PRG_Main"]},
       {"name": "SlowTask", "interval_ms": 200, "pous": ["PRG_Diag"]}
     ]
     ```

   - библиотеки — описание в `libraries` (название, компания, версия; формат — `libraries._doc` в
     `config.json` и [tools/README.md](../tools/README.md#библиотеки-library-и-deps)), ключ — в
     `modes.normal.libraries`. Standard уже подключена;
   - устройство — для сборки и тестов подходит Control Win V3 по умолчанию; устройство своего ПЛК —
     раздел 6.
5. Убрать тесты шаблона: они ссылаются на `FB_Example` и `F_Clamp`, которых после экспорта в `src/` нет.

   ```bash
   git rm -r tests/POUs/FB_Example_Test tests/POUs/FB_F_Clamp_Test
   ```

   В `tests/POUs/PRG_Tests/` удалить экземпляры `fbF_ClampTest` и `fbExampleTest` и их вызовы в
   `xAllFinished`, объявить экземпляр своего первого набора (раздел 5). Обоснования iec_checker и
   документы с примером — как в разделе 3, шаги 5 и 6.
6. Собрать проект из текстов и проверить компиляцию:

   ```bat
   python tools\codesys\cds.py import --clean
   python tools\codesys\cds.py build
   python tools\codesys\cds.py check --tests
   ```

   `import --clean` удаляет скопированный проект и создаёт `build\<имя>.project` заново — с устройством,
   задачами и библиотеками из `config.json` (без `--clean` импорт откажется, если устройство проекта
   не совпадает с `config.json`). Ошибки компиляции здесь обычно означают зависимость от того, что не
   перенеслось: вызов `ACTION`, графический POU, библиотеку без описания в `config.json`.
7. Проверить цикл «импорт → экспорт» и правила кода:

   ```bat
   python tools\codesys\cds.py roundtrip
   python tools\check\st_lint.py --strict
   python tools\check\run_iec_checker.py --tests
   ```

   `roundtrip` подтверждает, что импорт и экспорт воспроизводят `src/` и `tests/` побайтно; файл
   `*.pvl.st` даст расхождение — в чистом проекте объект не создаётся. На коде, написанном не по
   правилам шаблона, `st_lint` выдаст замечания: заголовки POU (STL001), табы (STL002), префиксы имён
   (STL006, STL007), «магические числа» (STL008). Исправьте код или ослабьте правила в
   `tools/check/st_lint.toml` (разделы `[severity]` и `[rules.STLxxx]`,
   [tools/README.md](../tools/README.md#checkst_lintpy--линтер)). Находки iec_checker исправьте или
   обоснуйте в `tools/check/iec_checker_allow.toml` и `docs/STATIC_ANALYSIS.md` — иначе CI не пройдёт.
8. Закоммитить `src/`, `tests/`, `tools/` и `docs/`; дальше — ежедневная работа (раздел 4).
   Визуализации и дерево устройств в этом цикле в git не попадают: их добавляют в IDE как ручной шаг, а
   проект, пересобранный с `--clean`, их не содержит.
