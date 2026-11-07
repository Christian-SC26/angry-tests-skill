# Глава 6. Fancy Tests — «модные» виды тестов

Идея главы: юнит- и интеграционные тесты — только часть автоматической проверки.
Сборка (CI) должна падать и от других видов проверок. Ниже — каждое правило из выдержек
([yegor256/at](https://github.com/yegor256/at)): инструменты, названные автором, и наши
дополнения для других экосистем (помечены «также»).

| № | Правило | Инструменты автора | Также |
|---|---------|--------------------|-------|
| 6.1 | Проверяй грамматику и опечатки (в коде, документации, сообщениях) | languagetool, typos-action | codespell, cspell, Vale |
| 6.2 | Применяй Property-Based Testing | jqwik, quickcheck | Hypothesis (Python), fast-check (JS), proptest (Rust), gopter / `testing/quick` (Go), SwiftCheck |
| 6.3 | Используй фаззинг | jqf, oss-fuzz, syzkaller | libFuzzer, AFL++, cargo-fuzz, `go test -fuzz`, Atheris (Python), Jazzer (JVM) |
| 6.4 | Используй мутационное тестирование | pitest | mutmut / cosmic-ray (Python), Stryker (JS/C#), cargo-mutants, go-mutesting, Mull (C++) |
| 6.5 | Проверяй раскладку файлов во время сборки | (jtcop для тестов) | свои скрипты/правила: все файлы на своих местах, нет лишнего |
| 6.6 | Нагрузочные тесты | jmeter, locust | k6, Gatling, wrk |
| 6.7 | Тесты производительности (микробенчмарки) | jmh | pytest-benchmark, criterion (Rust), `testing.B` (Go), Google Benchmark (C++), XCTest `measure` |
| 6.8 | Сравнивай производительность новой и предыдущей сборки | jmh-benchmark-action | github-action-benchmark, Bencher |
| 6.9 | Линтеры | eslint, rust-clippy, checkstyle, pylint, ruff | golangci-lint, rubocop, SwiftLint, qulice (Java, проект Бугаенко) |
| 6.10 | Статический анализ при сборке | clang-tidy, spotbugs, infer | PMD, Error Prone, mypy/pyright, CodeQL, Semgrep |
| 6.11 | Собирай метрики кода и роняй сборку при превышении порогов | — | цикломатическая сложность, размер классов, покрытие (JaCoCo с минимальным порогом), cohesion |
| 6.12 | Тестируй SQL-запросы | pgtap | tSQLt (MSSQL), utPLSQL (Oracle) |
| 6.13 | Находи самые медленные SQL-запросы | dexter | pg_stat_statements, `EXPLAIN ANALYZE` в тестах |
| 6.14 | Используй санитайзеры | — | ASan, UBSan, TSan, MSan (`-fsanitize=`), Rust `-Zsanitizer`, Go `-race` |
| 6.15 | Используй динамический анализ | — | Valgrind, Helgrind, DTrace, Go race detector |
| 6.16 | Ищи клоны кода | simian | PMD CPD, jscpd |
| 6.17 | Тестирование в браузере | selenium, playwright | Cypress, WebdriverIO |
| 6.18 | Тестирование в нескольких браузерах | saucelabs | BrowserStack, playwright с chromium/firefox/webkit |
| 6.19 | Кросс-браузерное тестирование | — | то же |
| 6.20 | GUI-тестирование | — | XCUITest, Espresso, Appium |
| 6.21 | Облачные анализаторы кода | SonarQube, Snyk, Codacy | CodeClimate, DeepSource |
| 6.22 | Live-тесты, проверяющие продакшен-конфигурации | — | тест, запускающий приложение с настоящей конфигурацией (по тегу/вручную) |
| 6.23 | Health checks после деплоя | — | `/health`-эндпоинт + проверка в пайплайне деплоя, smoke-тесты |
| 6.24 | Генерируй тесты при сборке | randoop | EvoSuite, Pynguin |
| 6.25 | Тесты безопасности | zaproxy (OWASP ZAP) | Burp, Trivy, OWASP Dependency-Check, bandit, gosec |
| 6.26 | Проверка лицензионной чистоты | ort, reuse-action | licensee, cargo-deny, license-checker |
| 6.27 | Находи избыточные тесты | — | анализ покрытия по тестам + мутационное тестирование: тесты, которые не убивают ни одного уникального мутанта |
| 6.28 | Проверяй архитектуру при сборке | archunit | import-linter (Python), dependency-cruiser (JS), cargo-deny/модульные правила |

## Как применять
- При настройке проекта предлагай минимальный набор: линтер (6.9), статический анализ (6.10),
  метрики с порогами (6.11), мутационное тестирование (6.4), проверку опечаток (6.1).
  Остальное — по природе проекта (веб → 6.17–6.19, БД → 6.12–6.13, C/C++ → 6.14–6.15,
  сервис → 6.6, 6.22, 6.23).
- Каждая проверка должна **ронять сборку**, а не просто печатать отчёт.
- Property-based (6.2) и фаззинг (6.3) — естественное продолжение принципов 2.7, 3.8, 3.9:
  они систематически ищут «неудобные» входы, которые человек не придумает.
- Мутационное тестирование (6.4) — главный ответ на *The Liar* и *Line Hitter*:
  покрытие показывает, что код выполнялся, а мутанты — что тесты действительно что-то проверяют.
