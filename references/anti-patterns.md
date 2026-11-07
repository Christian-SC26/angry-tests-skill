# Unit-testing anti-patterns and the Angry Tests rules that fix them

Source: Yegor Bugayenko, "Unit Testing Anti-Patterns, Full List" (2018),
https://www.yegor256.com/2018/12/11/unit-testing-anti-patterns.html. The post compiles anti-patterns
from Rob Fletcher (*Spock: Up and Running*), Boni Garcia (*Mastering Software Testing with JUnit 5*),
James Carr (*TDD Anti-Patterns*) and William E. Kempf. The "Rules" column maps each anti-pattern to the
book's rules (our mapping). In a review, name the anti-pattern and the rule number.

| Anti-pattern | What it is | Rules |
|--------------|------------|-------|
| **Cuckoo** (Stranger) | A test method that sits in a test class it does not belong to | 3.26 |
| **Test-per-Method** | One test per production method (one test class per production class is fine) | 3.26, 3.17 |
| **Anal Probe** | Unhealthy access, such as reading private fields through reflection | 2.13, 2.20 |
| **Conjoined Twins** | "Unit tests" that are integration tests with no isolation | 3.1, 3.20, 2.28 |
| **Happy Path** | Only expected results, no boundaries or exceptions | 3.5, 2.7, 2.17, 3.9 |
| **Slow Poke** | A unit test that runs extremely slowly | 3.1, 3.34, 3.15, 3.42 |
| **Giant** | A test thousands of lines long, often a sign of a God Object | 2.1, 2.30 |
| **Mockery** | So many mocks and stubs that the system under test goes untested | 2.28, 3.27, 3.28 |
| **Inspector** | Breaks encapsulation to reach 100% coverage and fails on every refactoring | 2.13, 2.16, 2.20 |
| **Generous Leftovers** (Chain Gang, Wet Floor) | One test leaves data that another test depends on | 2.10, 2.11, 2.27, 3.33 |
| **Local Hero** (Hidden Dependency, Environmental Vandal) | Depends on the author's environment and fails elsewhere | 3.9, 3.10, 3.23, 3.38 |
| **Nitpicker** | Compares the whole output when only a small part matters | 2.16, 3.6 |
| **Secret Catcher** | No assertions; the "check" is that no exception escaped | 2.3, 2.5 |
| **Dodger** | Many tests of minor side effects, none of the core behavior | 2.17, 2.24, 3.5 |
| **Loudmouth** | Fills the console with diagnostics and log output | 3.12 |
| **Greedy Catcher** | Catches exceptions and swallows the stack trace | 3.25, 2.6 |
| **Sequencer** | Depends on the order of items in an unordered collection | 2.16 |
| **Enumerator** (Test With No Name) | Names like `test1`, `test2` | 2.14 |
| **Free Ride** (Piggyback) | A new assertion added to an existing test instead of a new test | 2.2, 2.33 |
| **Excessive Setup** (Mother Hen) | Setup so large that you cannot see the check | 2.1, 2.10, 2.30, 3.46 |
| **Line Hitter** | Runs the code without analyzing the output | 2.3, 2.25, 6.4 |
| **Forty-Foot Pole Test** | Works at a distance through many layers of abstraction | 3.4, 2.13 |
| **The Liar** (Evergreen, Success Against All Odds) | Validates nothing and passes in every scenario | 2.3, 2.29, 6.4 |

## Quick diagnosis during review
- No `assert`, `expect` or matcher in the test: Secret Catcher or Line Hitter (2.3).
- `@BeforeEach`, class fields, `static final` data: Generous Leftovers or Excessive Setup (2.10, 2.11).
- `Mockito.mock`, `when(...)`, `verify(...)`, `MagicMock`, `jest.fn()` for domain dependencies: Mockery (2.28, 3.27).
- `setAccessible`, `getDeclaredField`, `_private`, `send(:private_method)`: Anal Probe or Inspector (2.13).
- `assertEquals(expectedWholeJson, actual)` when one field matters: Nitpicker (3.6).
- `catch (Exception e) {}` or `except: pass`: Greedy Catcher (3.25).
- `println`, `print` or `console.log` in a test: Loudmouth (3.12).
- `Thread.sleep`, `time.sleep` or `setTimeout` used to wait: Slow Poke and a flaky test (3.15).
- Hard-coded `localhost:8080`, `/tmp/test.txt` or real URLs: Local Hero (3.38, 3.10, 3.23).
- Every test uses "normal" inputs: Happy Path (3.5, 2.7).
