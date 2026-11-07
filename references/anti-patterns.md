# Антипаттерны юнит-тестирования и правила Angry Tests, которые их лечат

Источник списка: Yegor Bugayenko, «Unit Testing Anti-Patterns, Full List» (2018),
https://www.yegor256.com/2018/12/11/unit-testing-anti-patterns.html — сводка из книг Rob Fletcher
(*Spock: Up and Running*), Boni Garcia (*Mastering Software Testing with JUnit 5*), James Carr
(*TDD Anti-Patterns*) и William E. Kempf. Колонка «Правила» — сопоставление с принципами книги
(наше), используй её в ревью: называй антипаттерн и номер правила.

| Антипаттерн | Суть | Правила |
|-------------|------|---------|
| **Cuckoo** (Stranger) | Тестовый метод в чужом тестовом классе | 3.26 |
| **Test-per-Method** | Тест один-к-одному на каждый метод продакшен-кода (один-к-одному на класс — нормально) | 3.26, 3.17 |
| **Anal Probe** | Нездоровые способы: чтение приватных полей рефлексией | 2.13, 2.20 |
| **Conjoined Twins** | «Юнит-тесты», которые на деле интеграционные, без изоляции | 3.1, 3.20, 2.28 |
| **Happy Path** | Только ожидаемые результаты, без границ и исключений | 3.5, 2.7, 2.17, 3.9 |
| **Slow Poke** | Невероятно медленный юнит-тест | 3.1, 3.34, 3.15, 3.42 |
| **Giant** | Тест на тысячи строк — часто признак God Object | 2.1, 2.30 |
| **Mockery** | Столько моков/стабов, что тестируемая система не тестируется | 2.28, 3.27, 3.28 |
| **Inspector** | Нарушает инкапсуляцию ради 100% покрытия; ломается при рефакторинге | 2.13, 2.16, 2.20 |
| **Generous Leftovers** (Chain Gang, Wet Floor) | Один тест оставляет данные, от которых зависит другой | 2.10, 2.11, 2.27, 3.33 |
| **Local Hero** (Hidden Dependency, Environmental Vandal) | Зависит от окружения автора; у других падает | 3.9, 3.10, 3.23, 3.38 |
| **Nitpicker** | Сравнивает весь вывод, хотя важна малая часть | 2.16, 3.6 |
| **Secret Catcher** | Нет assert, «проверка» — что исключение не вылетело | 2.3, 2.5 |
| **Dodger** | Много тестов на мелкие побочные эффекты, ядро поведения не проверено | 2.17, 2.24, 3.5 |
| **Loudmouth** | Засоряет консоль диагностикой и логами | 3.12 |
| **Greedy Catcher** | Ловит исключения и глотает stack trace | 3.25, 2.6 |
| **Sequencer** | Зависит от порядка в неупорядоченной коллекции | 2.16 |
| **Enumerator** (Test With No Name) | Имена `test1`, `test2` | 2.14 |
| **Free Ride** (Piggyback) | Новая проверка дописана в существующий тест вместо нового | 2.2, 2.33 |
| **Excessive Setup** (Mother Hen) | Огромная подготовка, за которой не видно проверки | 2.1, 2.10, 2.30, 3.46 |
| **Line Hitter** | Тест просто выполняет код, не анализируя результат | 2.3, 2.25, 6.4 |
| **Forty-Foot Pole Test** | Тест действует «издалека» через множество слоёв | 3.4, 2.13 |
| **The Liar** (Evergreen, Success Against All Odds) | Ничего не проверяет и проходит всегда | 2.3, 2.29, 6.4 |

## Быстрая диагностика в ревью
- В тесте нет `assert`/`expect`/матчера → Secret Catcher / Line Hitter (2.3).
- Есть `@BeforeEach`, поля класса, `static final` данные → Generous Leftovers / Excessive Setup (2.10, 2.11).
- `Mockito.mock`, `when(...)`, `verify(...)`, `MagicMock`, `jest.fn()` для доменных зависимостей → Mockery (2.28, 3.27).
- `setAccessible`, `getDeclaredField`, `_private`, `send(:private_method)` → Anal Probe / Inspector (2.13).
- `assertEquals(expectedWholeJson, actual)` при важности одного поля → Nitpicker (3.6).
- `catch (Exception e) {}` или `except: pass` → Greedy Catcher (3.25).
- `println`/`print`/`console.log` в тесте → Loudmouth (3.12).
- `Thread.sleep`, `time.sleep`, `setTimeout` для ожидания → Slow Poke + плавающий тест (3.15).
- Захардкоженные `localhost:8080`, `/tmp/test.txt`, реальные URL → Local Hero (3.38, 3.10, 3.23).
- Все тесты на «нормальные» входы → Happy Path (3.5, 2.7).
