# Chapter 6. Fancy Tests

Unit and integration tests cover only part of automated verification. Other kinds of checks
should fail the build too. The table lists every rule from the excerpts
([yegor256/at](https://github.com/yegor256/at)), the tools the author names, and our additions
for other ecosystems in the "Also" column.

| # | Rule | Author's tools | Also |
|---|------|----------------|------|
| 6.1 | Test for grammar mistakes and typos (in code, docs, messages) | languagetool, typos-action | codespell, cspell, Vale |
| 6.2 | Employ Property Based Testing tools | jqwik, quickcheck | Hypothesis (Python), fast-check (JS), proptest (Rust), gopter or `testing/quick` (Go), SwiftCheck |
| 6.3 | Use Fuzzing tools | jqf, oss-fuzz, syzkaller | libFuzzer, AFL++, cargo-fuzz, `go test -fuzz`, Atheris (Python), Jazzer (JVM) |
| 6.4 | Use Mutation Testing | pitest | mutmut or cosmic-ray (Python), Stryker (JS/C#), cargo-mutants, go-mutesting, Mull (C++) |
| 6.5 | Check files layout during the build | (jtcop for tests) | custom scripts or rules: every file in its place, nothing extra |
| 6.6 | Use Load Tests | jmeter, locust | k6, Gatling, wrk |
| 6.7 | Use Performance Tests (microbenchmarks) | jmh | pytest-benchmark, criterion (Rust), `testing.B` (Go), Google Benchmark (C++), XCTest `measure` |
| 6.8 | Benchmark new vs. previous builds | jmh-benchmark-action | github-action-benchmark, Bencher |
| 6.9 | Use Linters | eslint, rust-clippy, checkstyle, pylint, ruff | golangci-lint, rubocop, SwiftLint, qulice (Java, a Bugayenko project) |
| 6.10 | Use Static Analysis at build time | clang-tidy, spotbugs, infer | PMD, Error Prone, mypy or pyright, CodeQL, Semgrep |
| 6.11 | Collect source code metrics and fail when thresholds are exceeded | none named | cyclomatic complexity, class size, coverage (JaCoCo with a minimum), cohesion |
| 6.12 | Test SQL queries | pgtap | tSQLt (MSSQL), utPLSQL (Oracle) |
| 6.13 | Detect slowest SQL queries | dexter | pg_stat_statements, `EXPLAIN ANALYZE` in tests |
| 6.14 | Use Sanitizers | none named | ASan, UBSan, TSan, MSan (`-fsanitize=`), Rust `-Zsanitizer`, Go `-race` |
| 6.15 | Use Dynamic Analysis tools | none named | Valgrind, Helgrind, DTrace, Go race detector |
| 6.16 | Employ Clone Detection tools | simian | PMD CPD, jscpd |
| 6.17 | Use In-Browser Testing | selenium, playwright | Cypress, WebdriverIO |
| 6.18 | Use Multi-Browser Testing | saucelabs | BrowserStack, playwright with chromium/firefox/webkit |
| 6.19 | Use Cross-Browser Testing | none named | same as 6.18 |
| 6.20 | Use GUI Testing | none named | XCUITest, Espresso, Appium |
| 6.21 | Use Cloud Code Analyzers | SonarQube, Snyk, Codacy | CodeClimate, DeepSource |
| 6.22 | Create Live Tests that verify production-ready configurations | none named | a test that boots the app with the real configuration (behind a tag or run by hand) |
| 6.23 | Use Health Checks after deployment | none named | a `/health` endpoint checked by the deploy pipeline, smoke tests |
| 6.24 | Generate tests at build time | randoop | EvoSuite, Pynguin |
| 6.25 | Employ Security Tests | zaproxy (OWASP ZAP) | Burp, Trivy, OWASP Dependency-Check, bandit, gosec |
| 6.26 | Do License Compliance Testing | ort, reuse-action | licensee, cargo-deny, license-checker |
| 6.27 | Detect redundant tests | none named | per-test coverage plus mutation testing: tests that kill no unique mutant |
| 6.28 | Validate architecture at build time | archunit | import-linter (Python), dependency-cruiser (JS), module rules in cargo-deny |

## How to apply
- When setting up a project, propose a minimal set first: a linter (6.9), static analysis (6.10),
  metrics with thresholds (6.11), mutation testing (6.4) and a typo check (6.1).
  Add the rest to fit the project: web apps get 6.17–6.19, databases 6.12–6.13,
  C and C++ 6.14–6.15, services 6.6, 6.22 and 6.23.
- Each check must **fail the build**. A report that nobody reads does not count.
- Property-based testing (6.2) and fuzzing (6.3) extend rules 2.7, 3.8 and 3.9:
  they search for the hostile inputs a person would not think of.
- Mutation testing (6.4) answers *The Liar* and *Line Hitter*. Coverage shows that code ran;
  killed mutants show that the tests check something.
