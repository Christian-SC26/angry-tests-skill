# Layout and naming of tests

Source: Yegor Bugayenko, "On the Layout of Tests" (2023),
https://www.yegor256.com/2023/01/19/layout-of-tests.html; rule 3.26 of the book; and
[jtcop](https://github.com/volodya-lombrozo/jtcop), which enforces these rules for Java.

## Two formal rules
1. **A test class holds only test methods.** No fields, no private static helpers, no nested
   utilities. (In Java: only methods annotated with `@Test` or `@ParameterizedTest`.)
2. **A test package holds only paired classes.** Each class with a `Test` or `ITCase` suffix maps
   one-to-one to a production class in the same package. Nothing else goes there.

## Names
- `Phrases.java` pairs with `PhrasesTest.java`. The test name tells the reader where to look:
  drop the suffix and you find the class.
- Methods are sentences about behavior: `countsSimpleGreetings()`, not `test1()` (rule 2.14).
- Class and method read as one phrase: "Phrases counts simple greetings".

## Structure (Maven/Gradle)
```
src/
  main/java/foo/
    Phrases.java
    Greetings.java
    FactoryOfPhrases.java        <- fakes and test-data factories live in production code
  test/java/foo/
    PhrasesTest.java
    GreetingsTest.java
    GreetingsITCase.java         <- integration test of one class
    FactoryOfPhrasesTest.java    <- the fake has its own tests
    it/
      SimpleGuessingITCase.java  <- integration scenario across several classes
```
- The `foo.it` package exists only in tests. It is the one allowed exception, for scenarios
  that belong to no single class.

## Unit vs. integration in Maven
- `*Test` runs in the `test` phase (Surefire). These tests are fast, and the build fails on the first red one.
- `*ITCase` runs under Failsafe: `pre-integration-test` (acquire resources, such as a MySQL instance),
  `integration-test` (run the tests, record results to a file), `post-integration-test` (release
  resources), `verify` (fail the build if any test is red).

## Where test prerequisites come from (most preferred first)
1. **Fake objects in production code** (`FactoryOfPhrases.aboutLondon()`, `FkStorage`). Other
   projects can reuse them, and they have their own tests.
2. **Test-framework extensions** (a JUnit `ParameterResolver` with `@ExtendWith`). Acceptable,
   but they couple to the whole suite and are hard to test in isolation.
3. **Private static helpers in the test class, or `FooUtils` utilities in `foo.support`.** Wrong:
   an anti-pattern.

## Other languages (adaptation)
| Language | File pair | Integration tests |
|----------|-----------|-------------------|
| Python | `pkg/phrases.py` with `tests/pkg/test_phrases.py` | `tests/it/...` plus `@pytest.mark.deep` |
| Go | `phrases.go` with `phrases_test.go` (same package) | build tag `//go:build integration` |
| JS/TS | `src/phrases.ts` with `test/phrases.test.ts` (or side by side) | `*.it.test.ts`, a separate Jest project |
| Ruby | `lib/foo/phrases.rb` with `test/foo/test_phrases.rb` | a separate rake task |
| Rust | `#[cfg(test)] mod tests` in the same file, or `tests/phrases.rs` | `tests/` plus `#[ignore]` for deep tests |
| Swift | `Sources/Foo/Phrases.swift` with `Tests/FooTests/PhrasesTests.swift` | a separate test target or test plan |
