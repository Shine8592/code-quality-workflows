# code-quality-workflows

**A skill for AI agents that treats code quality as a discipline, not a checklist — debugging, testing, review, refactoring, dogfooding, and fork maintenance, each with the failure modes that actually bite.**

[中文说明](README.zh-CN.md) · [Skill definition](SKILL.md) · [References](references/)

---

## Why this exists

Most "software quality" guidance for agents is a list of verbs: *write tests, do code review, refactor often*. That advice is unfalsifiable and therefore useless — nothing in it tells you what to do at 2 a.m. when a test passes locally and fails in CI, or when a reviewer says "this looks fine" and ships a regression.

This skill is organised around **failure modes**, not activities. Every section answers: what goes wrong here, how do I detect it early, and what does the fix cost if I detect it late?

```
Naive loop:      write code → run → looks fine → ship → incident
Disciplined loop: write code → predict failure → prove it fails → fix → prove it passes
```

The difference is not effort. It is knowing *where* to look.

---

## What's inside

| Area | Covers | Reference |
|---|---|---|
| **Debugging** | Python `debugpy` attach, Node `--inspect`, live Jupyter kernels — inspecting state instead of guessing | `python-debugpy-SKILL.md`, `node-inspect-debugger-SKILL.md`, `jupyter-live-kernel-SKILL.md` |
| **Testing** | Test-first as a design tool, not a coverage metric; the tests that are worth writing | `test-driven-development-SKILL.md` |
| **Review & QA** | Dogfooding your own product as a hostile user; issue taxonomy that separates bugs from opinions | `dogfood-SKILL.md`, `dogfood/issue-taxonomy.md` |
| **Refactoring** | Spike-then-rewrite; when a fork is cheaper than a patch; the shell-rewriting and brand-replacement traps | `spike-SKILL.md`, `foss-fork-and-rewrite/` |
| **Delivery** | Handing bug fixes to a human who will apply them by hand — diffs that survive being read once | SKILL.md § Delivering fix proposals |

---

## The three habits that do most of the work

**1. Make the bug reproducible before fixing it.**
A fix you cannot demonstrate failing first is a guess. The cost of writing the failing case is almost always lower than the cost of shipping a non-fix and being told "still broken".

**2. Prefer inspecting to printing.**
`print()` debugging works by bisection — each round halves the unknown space. A debugger or REPL jumps straight to the answer. The skill's references show the exact attach recipes for Python, Node, and Jupyter, because the friction of setting them up is what pushes people back to `print()`.

**3. Report the evidence, not the conclusion.**
"This should fix it" is not a delivery. "Here is the failing case, here is the run before the change, here is the run after, here is the one case I could not cover" is.

---

## Full-flow project audit

When asked to assess a whole codebase rather than fix one bug, the workflow is breadth-first and deliberately shallow at each step:

1. **Entry points** — what actually runs (not what the README claims)
2. **Boundaries** — where untrusted input enters, and what validates it there
3. **Error paths** — what happens when the network, disk, or a dependency fails
4. **Duplication** — the same logic implemented twice, differently, in two places
5. **Dead weight** — code that is never reached but must still be maintained

Only then look at style. Style findings are cheap to produce and expensive to act on; a missed boundary check is the reverse.

---

## Pitfalls this skill exists to prevent

- **Fixing the symptom the user reported** instead of the condition that produced it
- **Green tests as proof of correctness** when the tests never exercise the failing path
- **Refactoring while debugging** — two unknown variables, one measurement
- **Silent partial fixes** — the bug is gone on your machine because you changed something incidental
- **Review comments that cannot be acted on** ("consider improving error handling") — always name the input, the expected behaviour, and the evidence

---

## Installation

Drop the directory into your agent's skills folder:

```bash
git clone https://github.com/Shine8592/code-quality-workflows.git
cp -r code-quality-workflows ~/.hermes/skills/software-development/
```

The `SKILL.md` frontmatter (`name`, `description`) is what agent runtimes index; the `references/` tree is loaded on demand.

---

## Licence

MIT — see [LICENSE](LICENSE).
