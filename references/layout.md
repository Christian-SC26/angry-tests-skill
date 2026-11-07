# Раскладка и именование тестов

Источник: Yegor Bugayenko, «On the Layout of Tests» (2023),
https://www.yegor256.com/2023/01/19/layout-of-tests.html; правило 3.26 книги и инструмент
[jtcop](https://github.com/volodya-lombrozo/jtcop), который проверяет эти правила для Java.

## Два формальных правила
1. **В тестовом классе только тестовые методы.** Никаких полей, приватных статических
   хелперов, вложенных утилит. (В Java — только методы с `@Test`/`@ParameterizedTest`.)
2. **В тестовом пакете только классы-пары.** Каждый класс с суффиксом `Test` или `ITCase`
   отображается один-к-одному на класс продакшен-кода в том же пакете. Ничего другого.

## Имена
- `Phrases.java` → `PhrasesTest.java`. Имя теста — инструкция: убери суффикс и найдёшь класс.
- Методы — предложения о поведении: `countsSimpleGreetings()`, а не `test1()` (правило 2.14).
- Класс и метод вместе читаются как фраза: «Phrases counts simple greetings».

## Структура (Maven/Gradle)
```
src/
  main/java/foo/
    Phrases.java
    Greetings.java
    FactoryOfPhrases.java        ← фейки/фабрики тестовых данных живут в продакшен-коде
  test/java/foo/
    PhrasesTest.java
    GreetingsTest.java
    GreetingsITCase.java         ← интеграционный тест конкретного класса
    FactoryOfPhrasesTest.java    ← фейк тоже тестируется
    it/
      SimpleGuessingITCase.java  ← интеграционный сценарий через несколько классов
```
- Пакет `foo.it` существует только в тестах — это допустимое и единственное исключение
  для сценариев, не принадлежащих одному классу.

## Unit vs Integration в Maven
- `*Test` — фаза `test` (Surefire), быстрые, сборка падает сразу на красном тесте.
- `*ITCase` — Failsafe: `pre-integration-test` (поднять ресурсы, например MySQL) →
  `integration-test` (прогнать, результаты в файл) → `post-integration-test` (освободить
  ресурсы) → `verify` (уронить сборку, если что-то красное).

## Где брать предусловия теста (по убыванию предпочтения)
1. **Фейк-объекты в продакшен-коде** (`FactoryOfPhrases.aboutLondon()`, `FkStorage`) —
   переиспользуемы другими проектами, сами покрыты тестами.
2. **Расширения тестового фреймворка** (JUnit `ParameterResolver` + `@ExtendWith`) — допустимо,
   но они связаны со всем набором тестов и плохо тестируются отдельно.
3. **Приватные статические хелперы в тестовом классе, утилиты `FooUtils` в `foo.support`** —
   неправильно, антипаттерн.

## Другие языки (адаптация)
| Язык | Пара файлов | Интеграционные |
|------|-------------|----------------|
| Python | `pkg/phrases.py` ↔ `tests/pkg/test_phrases.py` | `tests/it/…` + `@pytest.mark.deep` |
| Go | `phrases.go` ↔ `phrases_test.go` (тот же пакет) | build tag `//go:build integration` |
| JS/TS | `src/phrases.ts` ↔ `test/phrases.test.ts` (или рядом) | `*.it.test.ts`, отдельный jest-проект |
| Ruby | `lib/foo/phrases.rb` ↔ `test/foo/test_phrases.rb` | отдельная задача rake |
| Rust | `#[cfg(test)] mod tests` в том же файле или `tests/phrases.rs` | `tests/` + `#[ignore]` для deep |
| Swift | `Sources/Foo/Phrases.swift` ↔ `Tests/FooTests/PhrasesTests.swift` | отдельный test target / test plan |
