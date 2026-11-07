# angry-tests

A [Claude Code](https://claude.com/claude-code) skill that writes and reviews automated tests by the rules of Yegor Bugayenko's book [*Angry Tests*](https://www.yegor256.com/angry-tests.html).

The skill covers the 110 rules Bugayenko published from the book in [yegor256/at](https://github.com/yegor256/at) (chapters 2, 3 and 6). It keeps his rule numbers, so a review comment reads "violates 2.10: shared `setUp`" and you can look the rule up in the source. Each rule has an explanation and a bad/good example. The examples use Java, Python, JS/TS, Go, Ruby, Rust and Swift.

## What you get

Claude Code loads the skill when you ask it to write or review tests ("review these tests against angry tests"), or when you type `/angry-tests`. With the skill loaded, Claude:

- writes tests with one assertion and a failure message, no shared state, fakes instead of mocks, hostile inputs and timeouts;
- reviews tests and reports each violation as `file:line`, rule number, problem and fix, grouped into critical, important and style.

## When to skip it

Bugayenko's rules are opinionated. They forbid mock frameworks, `setUp`/`tearDown`, shared constants and comments in tests. If your suite relies on Mockito-style mocks or shared fixtures, the skill will flag most of it. Install it when you want that pressure on your tests.

## Install

You need Claude Code. Clone the repository into your personal skills folder:

```bash
git clone https://github.com/Christian-SC26/angry-tests-skill.git ~/.claude/skills/angry-tests
```

To use it in one project only, clone into `.claude/skills/angry-tests` inside that project.

## What's inside

| File | Contents |
|------|----------|
| `SKILL.md` | Workflow for writing and reviewing tests, rules 2.1–2.33 and 3.1–3.49 with examples, checklist |
| `references/fancy-tests.md` | Chapter 6: rules 6.1–6.28 with tools for each ecosystem |
| `references/anti-patterns.md` | 23 unit-testing anti-patterns and the rules that fix them |
| `references/layout.md` | Test file layout and naming, `*Test` vs `*ITCase`, where fakes live |

## Coverage

Rule wordings match yegor256/at as of November 2026. Chapters 1, 4 and 5 of the book and its second volume (May 2026) are not public, so the skill leaves them out. The explanations and examples are mine, drawn from Bugayenko's blog posts. Where a rule is ambiguous, `SKILL.md` labels the explanation as an interpretation.

## License

MIT, see [LICENSE](LICENSE). The rule wordings are © 2025 Yegor Bugayenko and come from yegor256/at under the MIT license.
