---
name: angry-tests
description: Yegor Bugayenko's automated-testing rules from the book "Angry Tests" (yegor256). Use when writing, refactoring or reviewing unit and integration tests in the Angry Tests style - short tests, one assertion, failure messages, Hamcrest matchers, fakes instead of mocks, no setUp/tearDown, random and hostile inputs, fast/deep classification, timeouts, thread safety, fancy tests. Triggers - "angry tests", "Bugayenko", "yegor256", "review my tests", "write tests", "check tests against angry tests". Works for any language (Java, Python, JS/TS, Go, Ruby, Rust, Swift, C++).
---

# Angry Tests: Yegor Bugayenko's testing rules

*Angry Tests* is a book on automated testing by Yegor Bugayenko (@yegor256).
Volume 1 (April 2025) holds about 159 short recommendations, each about a page long, with examples in Java, C++, Go, Python, Ruby and JavaScript.
Volume 2 (May 2026) adds other kinds of tests, AI-assisted testing and the idea of "Headless Angry Tests".

This skill builds on the excerpts Bugayenko published from the book in
[yegor256/at](https://github.com/yegor256/at) (chapters 2, 3 and 6, with his rule numbers)
and on his blog, where he discusses many of the ideas at length. The rule wordings come from the excerpts.
The explanations, examples and resolutions of conflicts between rules are ours. Where it matters, this file labels them as interpretation.
Chapters 1, 4 and 5 and Volume 2 are not public. Do not invent their content, and do not cite rule numbers that do not appear below.

Reply in the language the user writes in.

Reference files (read when needed):
- `references/fancy-tests.md`: chapter 6, other kinds of automated checks and tools for each ecosystem.
- `references/anti-patterns.md`: Bugayenko's list of 23 unit-testing anti-patterns and the rules that fix each one.
- `references/layout.md`: layout and naming of test files, `*Test` vs `*ITCase`, where fakes live.

---

## 0. Philosophy: why tests should be angry

1. **A test hunts for bugs.** It does not demonstrate that the code works. A good test tries to break
   the code with boundary values, garbage input, Unicode, empty and huge data, concurrency,
   and a slow or hostile environment. A suite that checks only the happy path catches almost
   nothing (rules 3.5, 2.17, 3.9).
2. **A test is documentation that cannot lie.** The test name states a behavior of the object.
   The failure message explains what broke. Comments add nothing (2.14, 2.9, 2.15).
3. **A test checks the contract.** Use the public interface only: no reflection,
   no private methods, no `verify()` on internal calls (2.13, 2.16, 2.20, 3.27).
4. **Tests put pressure on design.** When you cannot write a short test, the object's design
   is at fault: mutable state, statics, hidden dependencies, god objects.
   Single-statement tests push production code toward immutable objects (2.30).
5. **A bad test beats no test** (2.32), and **every code change brings new tests** (2.33).
   An imperfect test still earns its place.
6. **Never forgive a red test.** Do not adjust an expectation to match a bug. A test either passes
   with an honest expectation or stays disabled with a reason (2.29, 2.19).

---

## 1. How to apply this skill

### Writing new tests
1. Identify the object's contract: what it promises through docs, signatures and types. Test only that.
2. List the pain points where this code will most likely break: boundaries, empty and huge inputs,
   Unicode, null/None, concurrent access, I/O errors, timeouts.
3. Write one test per pain point: a sentence for a name, its own irregular data,
   one assertion with a failure message, no shared fields, no `setUp`.
4. Replace dependencies with fakes (small but real implementations of the interface). Use the real
   file system inside a temporary directory. Use only local network servers on ephemeral ports.
5. Add timeouts and tags (fast/deep). Make sure the test logs nothing and passes without Internet access.
6. Run the test through the checklist in section 5.

### Reviewing tests
Report each violation as: file:line, rule number, the problem, the fix (with code).
Group findings by severity:
- **Critical**: the test checks nothing or lies. Missing assertion (2.3), swallowed exceptions
  (3.25), `assertDoesNotThrow` (2.5), an expectation bent to match a bug (2.29), dependence on order
  or shared data (2.10, 2.11), network calls (3.23), unbounded waits (3.14), `sleep` (3.15).
- **Important**: mocks and `verify` (2.28, 3.27), reflection and private methods (2.13, 2.20),
  missing failure messages (2.9), assertions on logs or error text (2.24, 3.24, 3.37), happy path only (3.5).
- **Style**: length (2.1), names (2.14), comments (2.15), `"foo"` data (2.7),
  inlined literals (2.12), abbreviated flags (3.30).

Do not rewrite tests wholesale unless asked: report first, then edit. Duplication in tests
ranks last (3.48), so do not start with it.

---

## 2. Chapter 2. Basics

### 2.1 Keep every test shorter than a dozen lines.
A test longer than 12 lines does setup that belongs in a fake, a fixture object
or a matcher. Long setup also signals a design problem in the object under test: too many
dependencies or mutable state.

### 2.2 Assert only once per test.
One test makes one claim about behavior. With five asserts, the first failure hides
the other four, and the test name cannot describe five claims. To check five things,
write five tests. Combine a complex check of one fact into a matcher (`allOf`, a custom matcher)
instead of a chain of asserts.
> How this fits with 2.23 and 2.25 (interpretation): "once" refers to the claim the test exists
> to make. When a test has to walk a multi-step scenario, it also checks intermediate results
> (see below). Such tests are the exception and usually belong to the "deep" group.

### 2.3 Assert in every test.
A test without an assertion ("call it and see that nothing throws") is the *Secret Catcher* or
*Line Hitter* anti-pattern. It inflates coverage and catches nothing. Even when you only care that
the call succeeds, assert on its result.

### 2.4 Finish every test with an assertion.
The last statement of a test is an assertion. Code after it is either useless, a hidden influence
on other tests (cleanup, see 2.27), or a sign that the assertion sits in the wrong place.

### 2.5 Don't use `assertDoesNotThrow`.
It asserts nothing about the result. Check what the object returned or how its state changed.
An exception fails the test on its own.
```java
// bad
assertDoesNotThrow(() -> new Report(data).save(path));
// good
new Report(data).save(path);
MatcherAssert.assertThat("report file must be non-empty", Files.size(path), Matchers.greaterThan(0L));
```

### 2.6 Don't fail explicitly using `fail()`.
`try { x(); fail(); } catch (E e) {}` is an old idiom full of traps: swallowed exceptions,
over-broad catch blocks. Use `assertThrows` or matchers (`Assertions.assertThrows(...)`,
`pytest.raises`, `expect(...).toThrow()`).

### 2.7 Use irregular input values, instead of `"foo"`.
`"foo"`, `"test"`, `1` and `42` are convenient values that almost never break code.
Pick data that provokes bugs: Unicode and emoji, leading and trailing spaces, quotes and special
characters, `\n`, `\t`, `\0`, very long strings, negative numbers, `Integer.MAX_VALUE`, empty collections.
```java
final String name = "Zoë O'Brien  李 😀\t";
```

### 2.8 Use different inputs in every test, don't reuse them.
Each test is a fresh attempt to find a bug, and repeated data narrows the search.
The same idea drives 2.11 (no shared constants) and 3.8 (randomizers).

### 2.9 Always write failure messages.
A programmer sees the message first when a test fails, usually in a CI log with no IDE at hand.
Describe the expected behavior instead of restating the code:
```java
MatcherAssert.assertThat(
    "counts only non-empty greetings, ignoring blank lines",
    new Greetings(text).count(),
    Matchers.equalTo(3)
);
```
Python: `assert_that(actual, equal_to(3), "counts only non-empty greetings")` (PyHamcrest)
or `assert actual == 3, "counts only non-empty greetings"`. JS: `expect(x, 'message')`
(chai) or `assertThat('message', x, equalTo(3))` (hamjest). Go: `t.Errorf("counts only ...: got %d", n)`.
Rust: `assert_eq!(n, 3, "counts only non-empty greetings")`.

### 2.10 Don't share any data between tests; never use `setUp` and `tearDown`.
Each test stands alone. A reader follows it top to bottom without jumping to `@BeforeEach`,
class fields or fixtures. Shared state makes tests depend on execution order (the *Generous Leftovers*
anti-pattern) and hides setup from the reader. A test class holds test methods and nothing else.
Forbidden: `@Before/@BeforeEach/@After/@AfterEach/@BeforeAll`, `setUp/tearDown` in unittest,
`beforeEach/afterEach` in Jest or Mocha, shared mutable pytest fixtures with module or session scope.
For common setup, write a fake or a fixture object (3.46) that each test creates itself.

### 2.11 Don't define constants (static literals) inside tests.
`private static final String NAME = "Jeff";` in a test class is shared state in disguise:
tests start using the same data (breaking 2.8), and the coupling between them becomes invisible.
Each test declares its own data locally.

### 2.12 Don't inline constants; assign them to variables first.
A literal that serves as both input and expectation lives in one local variable. The reader sees
that the expected value follows from the input, and two magic copies cannot drift apart.
```java
// bad
assertThat("greets", new Greeting("Anna").text(), containsString("Anna"));
// good
final String name = "Anna-Marie Ö";
assertThat("greeting must mention the name", new Greeting(name).text(), containsString(name));
```
> This does not conflict with 2.30: the single statement is a goal for the test's logic,
> and data declarations may precede it.

### 2.13 Don't bypass object interfaces, especially not via Reflection.
No reading of private fields, no `setAccessible(true)`, `Whitebox`, `obj._private`,
`instance_variable_get` or monkey-patching of internals. A test that breaks encapsulation
breaks on every refactoring (*Anal Probe*, *Inspector*). If you cannot check a behavior through
the interface, either the contract does not include it or the object lacks a method. Both are
design problems.

### 2.14 Name tests as full English sentences starting with a verb, e.g., `buildsHtmlPage`.
`buildsHtmlPage`, `rejectsNegativeAmount`, `readsFileWithUnicodeName`. The class name is the subject:
"`HtmlPage` builds HTML page". No `test` or `should` prefixes, no numbers (`test1` is the
*Enumerator* anti-pattern). Python: `test_builds_html_page` (the runner needs the `test_` prefix;
the rest is a sentence). Go: `TestBuildsHtmlPage`. Rust: `builds_html_page`.
JS: `it('builds HTML page', ...)`.

### 2.15 Never write any comments inside or outside of test methods.
A test that needs a comment is unclear. Rewrite it, and improve its name and failure message.
Comments go stale, while names and messages run with every build.

### 2.16 Don't assert on behavior not promised by the object's contract.
Iteration order of a `HashSet`, the exact text of `toString()`, the wording of an exception message,
the number of internal calls and timings are details. Tests that pin them break on safe changes
and teach the team to ignore red builds (*Nitpicker*, *Sequencer*).

### 2.17 Assert on the most vulnerable pain points.
Test first where a bug is most likely and most expensive: boundaries, overflow, empty and huge
inputs, encodings, concurrency, failures of external resources. Checks of the obvious bloat
the suite and catch little.

### 2.18 Don't test getters, setters, and similar primitive functionality.
Trivial code does not break, and tests of real behavior cover it anyway. (Bugayenko would rather
have no getters and setters at all, but that is a design question.)

### 2.19 Don't delete tests, disable them instead.
A deleted test takes its knowledge of a scenario and a bug with it. A disabled test
(`@Disabled("...")`, `@pytest.mark.skip(reason=...)`, `it.skip`, `#[ignore = "..."]`, `t.Skip("...")`)
stays visible as a reminder and comes back with one edit. Always state the reason and link the ticket.

### 2.20 Never test private or protected object methods.
They are implementation details, and tests reach them through the public interface. A private method
complex enough to deserve its own test wants to become a separate object.

### 2.21 Close resources after use, such as files and sockets.
Use `try-with-resources`, `with`, `defer`, `ensure` or `Drop`. Leaked resources cause flaky failures
in neighboring tests and ports that stay busy.

### 2.22 Don't write feature code that is only used in tests.
`@VisibleForTesting`, `reset()`, setters "for tests", `isTestMode` flags and public fields added
for assertions let tests damage the design. If a test needs something, make it a proper part of
the contract or rethink the test.
> Nuance (interpretation, from Bugayenko's blog): he treats fake objects shipped with a library
> (`FkUser`, `FakeStorage`) as part of its API. Library clients use them in their own tests,
> so they do not count as test-only code.

### 2.23 Assert at every step while getting to the end of a use case.
When a test walks a multi-step use case, check the result of each step, not only the last one.
Otherwise a failure does not tell you which step went wrong, and later steps can mask the error.

### 2.24 Don't assert on side effects, like logs.
Logs are not part of the contract. A check that "the log contains X" ties the test to wording
and proves nothing about behavior (*Dodger*). See also 3.37.

### 2.25 Assert on all possible intermediate results.
Close to 2.23: when an object returns intermediate values (statuses, partial results, identifiers),
check that they are correct, not merely non-null.

### 2.26 Don't test constructors, they are code-free anyway.
Bugayenko holds that [constructors must be code-free](https://www.yegor256.com/2015/05/07/ctors-must-be-code-free.html):
they only assign attributes, so there is nothing to test. If a constructor contains logic, fix the
constructor and move the logic into methods that run on demand.

### 2.27 Clean up before a test, not after it.
A test prepares a clean environment at its start: it deletes and recreates its directory or empties
its table. Cleanup after a test does not run when the test crashes or the process dies, and it destroys
the evidence you need for debugging (see 3.36). A unique temporary directory per test works best (3.10).

### 2.28 Don't use mock frameworks, build fake objects instead.
Mocks (Mockito, unittest.mock, sinon, gomock) describe expected calls, which is implementation,
not behavior. Tests built on them are brittle and verbose, and they often test the mocks
themselves (*Mockery*). A [fake object](https://www.yegor256.com/2014/09/23/built-in-fake-objects.html)
is a small but honest implementation of the interface: an in-memory store, a fixed clock,
a local HTTP server. You write a fake once, test it, and reuse it.
```java
// bad
final Storage storage = Mockito.mock(Storage.class);
Mockito.when(storage.read("k")).thenReturn("v");
// good
final Storage storage = new FkStorage(new MapOf<>("k", "v"));
```

### 2.29 Don't forgive incorrect behavior, disable the test instead.
When code misbehaves, do not change the expectation to match ("let it be 4 instead of 5 for now").
Keep the honest expectation, disable the test with a link to the bug (2.19), and file the bug.
An adjusted expectation turns the test into *The Liar*.

### 2.30 Aim for one-statement tests.
In [single-statement unit tests](https://www.yegor256.com/2017/05/17/single-statement-unit-tests.html)
the body is one `assertThat(...)` that creates and calls the object inside it.
Algorithmic setup moves into objects and matchers you can reuse. Tests become declarative,
and production code drifts toward immutability.
```java
@Test
void countsWordsInUnicodeText() {
    MatcherAssert.assertThat(
        "counts words separated by any whitespace",
        new Words("Grüße,\tмир   李").count(),
        Matchers.equalTo(3)
    );
}
```

### 2.31 Use Hamcrest.
Matchers give readable checks, clear failure output ("Expected: ... but: ..."),
composition (`allOf`, `hasItem`, `not`) and custom matchers (3.3). Equivalents:
Java: Hamcrest (Bugayenko's choice). Python: PyHamcrest. JS/TS: hamjest.
C++: gMock matchers in GoogleTest. Go: github.com/corbym/gocrest. Rust: hamcrest2 or googletest-rust.
Ruby: RSpec matchers, or minitest with clear messages. Swift: Nimble.

### 2.32 Don't be discouraged from writing bad tests; they're better than nothing.
Write the test even when it comes out imperfect. You can improve it later, and a missing test
catches nothing. In a review, point out violations but accept a valuable test instead of
demanding a full rewrite.

### 2.33 Every time you change the code, add more tests.
Every change, a bug fix above all, comes with new tests. A bug fix starts with a test that
reproduces the bug (3.4, 3.5).

---

## 3. Chapter 3. Advanced

### 3.1 Classify tests as "fast" (50ms each) and "deep" (integrating everything).
Fast tests run on every build and every save. Deep tests (a real database, Docker, a browser,
external processes) run in a separate phase. Separate them with tags (3.20) and/or suffixes
(`*Test` / `*ITCase`, see `references/layout.md`).

### 3.2 Make tests flaky and unstable, then expect bugs to be reported and fixed.
(Interpretation) Tests should inject variation on purpose: random data (3.8), random order,
parallel runs (3.33), a hostile environment (3.9). Such a test fails now and then, and each failure
is a found bug, not "a flaky test, rerun it". Report the failure as a bug and fix it
(after investigation, see 3.41).

### 3.3 Create custom matchers and reuse them in assert statements.
A recurring check ("valid XML with node X", "JSON has field Y") becomes a matcher with a clear
description. Matchers make single-statement tests possible (2.30) and produce good failure messages.
```java
MatcherAssert.assertThat("renders a valid page", page, new IsValidHtml());
```

### 3.4 Reproduce bugs with the minimum possible scaffolding.
A bug test uses the smallest input and the fewest objects that still fail. If one object
reproduces the bug, do not boot the whole application.

### 3.5 Tests must reproduce particular bugs, not successful usage scenarios.
A test exists to show that a specific failure cannot happen. Before writing it, ask "what bug could
live here?" and write the test that would catch it. A suite of successful scenarios only is the
*Happy Path* anti-pattern.

### 3.6 Don't assert on unimportant details, don't be pedantic without reason.
Comparing the whole output when one part matters (*Nitpicker*) makes tests brittle.
Check what the contract cares about and match the rest with `containsString`, `hasEntry`,
XPath or JSONPath.

### 3.7 Invent a DSL and write test stories using it.
When you have many similar tests, describe them in a declarative format (for example, YAML files,
each a story of input and expectation) and let one parameterized test run all the stories.
Adding a case takes no code, and non-programmers can read the stories.

### 3.8 Use randomizers to generate test data.
Random strings, numbers, names and identifiers widen the search on every run. A failure must
print the seed or the values so you can reproduce it.
Tools: Java: `RandomStringUtils`, Instancio, EasyRandom, Datafaker. Python: Faker, Hypothesis.
JS: @faker-js/faker. Go: `math/rand` with `testing/quick`. Rust: `fake`, `proptest`.

### 3.9 Don't help your tests; create the most inconvenient environment for them.
Spaces and Unicode in paths, an unusual locale and time zone, read-only directories,
random test order, parallel runs, slow fakes, low memory limits. Code that survives this
environment will survive production.

### 3.10 Don't keep temporary files next to the source code; use a temporary directory.
`@TempDir`, `tmp_path` (pytest), `t.TempDir()` (Go), `tempfile`, `Dir.mktmpdir`, `tempfile::TempDir`.
Tests create no files in `src/` or the repository root.

### 3.11 Don't test abstract classes.
Test the concrete implementations. An abstract class has no behavior of its own, and testing it
through an anonymous stand-in tests the stand-in.

### 3.12 Don't let tests log anything; keep the log empty.
Tests stay silent (*Loudmouth*). No `System.out.println`, `print` or `console.log` in tests.
Noise hides real warnings. Configure production logging to stay quiet during tests where you can.

### 3.13 Parameterize tests.
One contract over many inputs is one parameterized test (`@ParameterizedTest`,
`@pytest.mark.parametrize`, table-driven tests in Go, `test.each` in Jest, `rstest`).
Each parameter set shows up as its own case in the report.

### 3.14 Stop tests on timeout; don't let them run forever.
Every test (above all those with I/O, threads or network) has a timeout: `@Timeout`,
`junit.jupiter.execution.timeout.default`, `pytest-timeout`, `jest --testTimeout`,
`go test -timeout`. A hanging test is worse than a failing one: it blocks CI and reports nothing.

### 3.15 Don't sleep for an arbitrary number of seconds; instead, wait for an event.
`Thread.sleep(1000)` either slows the suite or, on a slow machine, makes it flaky.
Wait for a condition: `CountDownLatch`, `Future.get`, Awaitility (`await().until(...)`),
polling with a timeout, `asyncio.Event`, channels in Go.

### 3.16 When it's necessary to simulate a hang, sleep for a billion seconds.
When a fake has to hang (to test the caller's timeouts), it sleeps for an obviously infinite time
(`TimeUnit.SECONDS.sleep(1_000_000_000L)`), not "a while" like 5 seconds. The test stays
deterministic: it checks the reaction to a hang instead of a race between two timers.

### 3.17 Let different tests test the same part of the feature code.
Overlap between tests is fine and does not count as duplication. Different tests approach the code
from different sides and catch different bugs. Do not delete a test because "that line is already covered".

### 3.18 Test whether your objects are thread-safe.
Objects that several threads may use get a concurrency test: many threads call the method at once,
then the test checks an invariant. Details:
[Java](https://www.yegor256.com/2018/03/27/how-to-test-thread-safety.html),
[Ruby](https://www.yegor256.com/2018/11/06/ruby-threads.html).
```java
@Test
void incrementsCounterFromManyThreads() throws Exception {
    final Counter counter = new Counter();
    final int threads = Runtime.getRuntime().availableProcessors() * 4;
    final CountDownLatch start = new CountDownLatch(1);
    final ExecutorService service = Executors.newFixedThreadPool(threads);
    final Collection<Future<?>> futures = new ArrayList<>(threads);
    for (int idx = 0; idx < threads; ++idx) {
        futures.add(service.submit(() -> { start.await(); counter.increment(); return null; }));
    }
    start.countDown();
    for (final Future<?> future : futures) { future.get(1L, TimeUnit.MINUTES); }
    MatcherAssert.assertThat("counts every increment exactly once", counter.value(), Matchers.equalTo(threads));
}
```
(This test runs past 12 lines. Rule 3.49 and libraries such as
[yegor256/threads](https://github.com/yegor256/threads) or cactoos `RunsInThreads` shrink it to one statement.)

### 3.19 Let tests retry when the behavior they are testing is flaky.
When the behavior under test is unstable by nature (an external service in a live test, a network
race, a probabilistic algorithm), a retry is acceptable (`@RetryingTest` from junit-pioneer,
`pytest-rerunfailures`, `jest.retryTimes`). Do not use retries to hide a bug in your own code (see 3.41).

### 3.20 Use tags to classify tests.
`@Tag("fast")`, `@Tag("deep")`, `@pytest.mark.slow`, build tags in Go, `#[ignore]` plus a separate
run in Rust. Tags let you run subsets: fast tests locally, deep tests in CI.

### 3.21 Let your testing framework repeat some tests to increase the chance of hitting the bug.
`@RepeatedTest(100)`, `pytest-repeat`, `go test -count=100`. Use repetition for concurrent code
and randomized tests in particular: a rare bug shows up on the hundredth run.

### 3.22 In tests for thread-safety, utilize all available CPUs.
Derive the thread count from `availableProcessors()` (`os.cpu_count()`, `runtime.NumCPU()`) or more.
Two threads rarely expose a race.

### 3.23 Run all tests with no Internet connection; they must pass.
Turn off the network and run the suite: it must pass. Replace external services with fakes or
local servers (Testcontainers, a local HTTP server on an ephemeral port). A test that depends on the
Internet is slow, flaky and hard to reproduce (*Local Hero*).

### 3.24 Don't assert on the details of errors.
Check the exception type or the fact of failure, not the message text. Wording changes,
and the contract does not include it (2.16).
```java
Assertions.assertThrows(IllegalArgumentException.class, () -> new Amount(-1));
```

### 3.25 Keep the scope of try/catch as small as possible.
A wide `try` catches exceptions from the wrong line, and an empty `catch` or a catch of the generic
`Exception` hides bugs (*Greedy Catcher*). Tests rarely need try/catch at all:
use `assertThrows` and let unexpected exceptions fail the test.

### 3.26 Strictly one test per feature file: [jtcop](https://github.com/volodya-lombrozo/jtcop).
`Foo.java` pairs with `FooTest.java` in the same package. A test file without a production
counterpart, or with an unclear name, is forbidden. jtcop checks this automatically.
See `references/layout.md`.

### 3.27 Don't use `verify()` from a mock framework.
`verify(mock).save(x)` checks *how* the object did its job, not *what* came out. Any refactoring
(a cache, batching, a different call order) breaks the test while the behavior stays correct.
Check the fake's state after the action: `assertThat(storage.read("k"), equalTo("v"))`.

### 3.28 Don't use PowerMock or similar frameworks.
Replacing statics, constructors and final classes through bytecode tricks points to a design problem:
static dependencies, `new` inside logic. Fix the design with constructor injection,
interfaces and fakes.

### 3.29 In tests, don't instantiate objects or call their methods with default arguments.
Pass every argument explicitly. Otherwise the test depends on default values, and when a default
changes, the test changes meaning while its code stays the same. Explicit arguments also document
what the test checks.

### 3.30 Don't abbreviate; use `curl --silent` instead of `curl -s`.
Test scripts, CI configs and commands use long flag forms. The reader should not have to recall
what `-s` means for a given tool.

### 3.31 Don't let feature objects do the job of tests; verify inputs and state.
(Interpretation) The test checks what matters itself. Do not rely on production code to throw
somewhere inside on bad data or to validate its own state. The test states that the inputs were
accepted and that the state after the action is correct.

### 3.32 Use decorating invariants to catch improper use of objects during testing.
Wrap a dependency in a decorator that enforces usage rules during the test: the resource closes
exactly once, nobody calls a method after `close()`, no calls come from a foreign thread, inputs
are not null. The decorator throws on a violation, and the test fails at the point of misuse,
without mocks or `verify`.
```java
final Input input = new StrictInput(new InputOf(file)); // throws on a second close() or on reading after close()
```

### 3.33 Run tests in parallel threads: [threads](https://github.com/yegor256/threads).
Enable parallel execution (`junit.jupiter.execution.parallel.enabled=true`,
`pytest -n auto` with pytest-xdist, `t.Parallel()`, Jest workers, cargo test by default).
Parallel runs speed up the suite and expose hidden shared state between tests.

### 3.34 Every fast test must take less than 100 milliseconds.
A hard limit for fast tests (aim for about 50 ms, see 3.1). Move a slow "fast" test to the deep
group or rewrite it (*Slow Poke*).

### 3.35 Don't mock the file system.
Use the real file system inside a temporary directory (3.10). A file-system mock tests the mock,
while the bugs (permissions, file-name encodings, symlinks, spaces in paths) live in the real one.

### 3.36 Make the place with temporary files accessible after the end of the test suite: [mktmp](https://github.com/yegor256/mktmp).
Create temporary files in a known place inside the build directory (`target/`, `build/`) instead of
the system `/tmp`, which gets wiped, so you can inspect what a test left behind after a CI failure.
This pairs with 2.27: clean up before the test.

### 3.37 Don't assert on the content of logs generated by feature code during tests.
Extends 2.24: capturing the logger and checking its lines ties the test to wording and log levels.
Check observable behavior instead.

### 3.38 Use ephemeral TCP ports.
No hard-coded `8080`: parallel tests and other processes collide on it. Ask the OS for a free port
(`new ServerSocket(0)`, `bind(('', 0))`, `:0` in Go, `port: 0` in Node).

### 3.39 Don't use inheritance to reuse test tools.
No `AbstractDatabaseTest` for test classes to extend: it brings implicit shared state and a brittle
hierarchy. Reuse through composition: fakes, matchers, fixture objects, JUnit extensions,
test libraries (3.49).

### 3.40 Don't be scared of long test classes, they are OK.
A test class is a set of independent scenarios, not an object with behavior. Bugayenko considers
5,000 lines in a test class acceptable. Do not split test classes artificially (that would also break 3.26).

### 3.41 Don't fix feature code in response to a flaky test failure.
Investigate a flaky test first: reproduce it deterministically (3.4, or with repetition, 3.21) and find
the cause in the test, the environment or the code. A blind fix to production code masks the real cause.

### 3.42 Use [maybeslow](https://github.com/yegor256/maybeslow) or a similar library to diagnose long-running tests.
maybeslow reports which test is running long while it still runs. Equivalents:
`pytest --durations=20`, `go test -v` with `-timeout`, `jest --detectOpenHandles`,
duration reports in Surefire.

### 3.43 Kill long-running tests on timeout.
Set a timeout for each test (3.14) and for the whole CI run (`timeout-minutes` in GitHub Actions,
`forkedProcessExitTimeoutInSeconds` in Surefire). CI must kill a process that hangs.

### 3.44 Inline fixtures instead of keeping them in fixture files.
Small inputs (JSON, XML or CSV of a dozen lines) go into the test as text blocks or multiline
strings. The reader sees the input and the expectation in one place without opening
`src/test/resources`.

### 3.45 Don't keep large fixtures in static files; let tests generate them.
A megabyte file in the repository is opaque and does not show which part matters. Generate large
data in code: a million rows, deep nesting, a huge file.

### 3.46 Create fixture objects that generate large fixtures at runtime.
Wrap the generation from 3.45 in objects: `new LargeCsv(rows, columns)`, `new DeepJson(depth)`.
You can reuse and test them, and they keep tests to one statement.

### 3.47 Keep reference fixtures as static files in the repository.
The exception to 3.44 and 3.45: real samples you cannot or should not generate (customer files,
responses from third-party APIs, golden outputs, files that once broke production).
Store them as-is and let the tests run all of them.

### 3.48 Code duplication in tests is the last problem to fix.
Each test must read on its own. Some repetition beats shared helper methods, fields and base classes
that couple tests together. DRY ranks lowest in tests.

### 3.49 Extract test libraries.
When fakes, matchers and fixture objects serve several modules or projects, move them into a
separate library (as Bugayenko did with threads, mktmp and maybeslow). This is the right alternative
to inheritance (3.39) and static utilities.

---

## 4. Chapter 6. Fancy Tests, in brief
Unit and integration tests are not enough. The build should also run other automated checks:
spelling, property-based tests, fuzzing, mutation testing, file-layout checks, load tests and
benchmarks (compared against the previous build), linters, static and dynamic analysis, metrics
with thresholds, SQL tests, sanitizers, clone detection, browser and cross-browser tests, cloud
analyzers, live tests of production configuration, health checks after deployment, test generation,
security tests, license compliance, redundant-test detection and architecture checks. The full list
6.1–6.28 with tools for each language is in `references/fancy-tests.md`.

---

## 5. Checklist for every test

- [ ] The name is a sentence that starts with a verb, with no `test`/`should` prefix or numbers (2.14)
- [ ] 12 lines or fewer, ideally one statement (2.1, 2.30)
- [ ] One assertion, placed last (2.2, 2.4), with a failure message (2.9)
- [ ] No `assertDoesNotThrow`, `fail()` or wide try/catch (2.5, 2.6, 3.25)
- [ ] Irregular or random data, unique to this test, held in local variables (2.7, 2.8, 2.11, 2.12, 3.8)
- [ ] No class fields, `setUp`/`tearDown` or test-class inheritance (2.10, 3.39)
- [ ] Public interface only, with no reflection and no private methods (2.13, 2.20)
- [ ] Checks the contract, not details: no logs, error text or order of unordered collections (2.16, 2.24, 3.24, 3.37)
- [ ] Fakes instead of mocks, no `verify`, no PowerMock (2.28, 3.27, 3.28)
- [ ] Real file system in a temporary directory, ephemeral ports, no Internet (3.10, 3.35, 3.38, 3.23)
- [ ] Has a timeout and no `sleep` (except the billion-second sleep inside a fake) (3.14, 3.15, 3.16)
- [ ] Closes its resources and cleans up at the start, not the end (2.21, 2.27)
- [ ] Logs nothing (3.12); a fast test runs under 100 ms, otherwise it carries the deep tag (3.1, 3.34, 3.20)
- [ ] No comments (2.15); explicit arguments (3.29); long flags in commands (3.30)
- [ ] Targets a specific bug or pain point instead of replaying the happy path (2.17, 3.5)

Project-level checklist:
- [ ] One test file per production file, same package, `Test`/`ITCase` suffix (3.26)
- [ ] The suite passes without network access and in parallel (3.23, 3.33)
- [ ] Timeouts exist per test and for the whole CI run (3.14, 3.43)
- [ ] Disabled tests carry a reason and a bug link, and nobody deleted tests (2.19, 2.29)
- [ ] Recurring checks are custom matchers, recurring setup is fakes and fixture objects (3.3, 3.46)
- [ ] The build runs linters, static analysis and mutation testing (6.4, 6.9, 6.10)

---

## 6. Full example: before and after

Before (a typical test with many violations):
```java
public class UserServiceTest {
    private static final String NAME = "foo";            // 2.11, 2.7
    private UserService service;
    private UserRepository repo;

    @BeforeEach                                           // 2.10
    public void setUp() {
        repo = Mockito.mock(UserRepository.class);        // 2.28
        service = new UserService(repo);
    }

    @Test
    public void testCreate() {                            // 2.14
        // create user                                     // 2.15
        User u = service.create(NAME);
        assertNotNull(u);                                 // 2.2, 2.9
        assertEquals("foo", u.name());                    // 2.12
        Mockito.verify(repo).save(Mockito.any());         // 3.27
        System.out.println("created " + u);               // 3.12
    }
}
```

After:
```java
final class UsersTest {
    @Test
    void storesCreatedUserWithUnicodeName() {
        final String name = "Zoë O'Brien 李 🦔";
        final Users users = new Users(new FkStorage());
        MatcherAssert.assertThat(
            "finds the created user by its exact unicode name",
            users.create(name).id(),
            Matchers.equalTo(users.byName(name).id())
        );
    }

    @Test
    void rejectsBlankName() {
        Assertions.assertThrows(
            IllegalArgumentException.class,
            () -> new Users(new FkStorage()).create(" \t\n"),
            "refuses a name made of whitespace only"
        );
    }
}
```

The same style in Python (pytest with PyHamcrest):
```python
from hamcrest import assert_that, equal_to

def test_stores_created_user_with_unicode_name():
    name = "Zoë O'Brien 李 🦔"
    users = Users(FakeStorage())
    assert_that(users.create(name).id, equal_to(users.by_name(name).id), "finds the created user by its exact unicode name")

def test_rejects_blank_name():
    with pytest.raises(ValueError):
        Users(FakeStorage()).create(" \t\n")
```

---

## 7. Related sources by Bugayenko
- The book: https://www.yegor256.com/angry-tests.html
- Excerpts (chapters 2, 3, 6): https://github.com/yegor256/at
- Unit Testing Anti-Patterns, Full List (2018): https://www.yegor256.com/2018/12/11/unit-testing-anti-patterns.html
- On the Layout of Tests (2023): https://www.yegor256.com/2023/01/19/layout-of-tests.html
- Single Statement Unit Tests (2017): https://www.yegor256.com/2017/05/17/single-statement-unit-tests.html
- Built-in Fake Objects (2014): https://www.yegor256.com/2014/09/23/built-in-fake-objects.html
- Constructors Must Be Code-Free (2015): https://www.yegor256.com/2015/05/07/ctors-must-be-code-free.html
- Thread safety: https://www.yegor256.com/2018/03/27/how-to-test-thread-safety.html
