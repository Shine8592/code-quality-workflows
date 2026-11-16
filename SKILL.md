---
name: code-quality-workflows
description: "Class-level software quality workflows: debugging, testing, code review, refactoring, QA dogfooding, fork/rewrite, and live Python kernels."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [debugging, testing, review, refactoring, qa, live-kernel, fork-rewrite]
    absorbed_skills: [dogfood, foss-fork-and-rewrite, node-inspect-debugger, python-debugpy, requesting-code-review, simplify-code, spike, systematic-debugging, test-driven-development, jupyter-live-kernel]
---

# Code Quality Workflows

Use this umbrella skill for class-level software development quality workflows: debugging, testing, code review, refactoring, exploratory QA, fork/rewrite, and live Python kernels. It absorbs narrower development-process skills into one broad class-level skill with labeled subsections.

## When to Use

Use this skill when the user asks to:

- Debug Python or Node.js code with debugpy or the Node inspector.
- Run or design systematic debugging sessions.
- Write tests first, run TDD cycles, or improve test coverage.
- Request or perform code review with security and quality gates.
- Simplify or clean up recent code changes.
- Run exploratory QA/dogfooding and write issue reports.
- Audit an entire local project end-to-end ("跑一遍所有流程、所有环节，看有没有没发现的 bug") and deliver a revision spec.
- Fork and rebrand open-source tools or scripts.
- Use a live Jupyter kernel for iterative Python exploration.
- Run a throwaway spike to validate an idea before building.

## Absorbed skill map

### `dogfood`
Use for exploratory QA, bug discovery, evidence capture, and report writing. Preserve `references/dogfood-SKILL.md` and `templates/dogfood/dogfood-report-template.md`.

### `foss-fork-and-rewrite`
Use for forking and rebranding open-source CLI tools/scripts. Preserve `references/foss-fork-and-rewrite-SKILL.md` and `templates/foss-fork-and-rewrite/fork-file-comparison-template.md`.

### `node-inspect-debugger`
Use for debugging Node.js via `--inspect` and Chrome DevTools Protocol CLI.

### `python-debugpy`
Use for debugging Python with pdb REPL and debugpy remote DAP.

### `requesting-code-review`
Use for pre-commit review: security scan, quality gates, and auto-fix.

### `simplify-code`
Use for parallel 3-agent cleanup of recent code changes.

### `spike`
Use for throwaway experiments to validate an idea before build.

### `systematic-debugging`
Use for 4-phase root-cause debugging: understand before fixing.

### `test-driven-development`
Use for TDD: enforce RED-GREEN-REFACTOR and tests before code.

### `jupyter-live-kernel`
Use for iterative Python via a live Jupyter kernel.

## Workflow

1. **Classify the task**: debug, test, review, refactor, QA, fork/rewrite, live exploration, or spike.
2. **Choose the subsection** and load the corresponding support files.
3. **Reproduce or inspect** before changing code.
4. **Apply the smallest safe change** or generate the requested artifact.
5. **Verify** with tests, lint, health checks, or a focused regression probe.
6. **Report** what changed, what passed, and any remaining risk.

## Full-flow project audit

Trigger: the user asks for **every flow and every step** to be exercised to find undiscovered bugs. This is an audit, not a fix session — deliver findings, not commits.

1. **Inventory from the filesystem first.** Enumerate every script, reference doc, template and test case with `ls`/`grep` and build the checklist from that list, never from memory. A doc that no other doc references is itself a finding (orphan capability).
2. **Cheap gates before expensive ones:** `python -m py_compile` on every script, then `--help` on every script. Catches argument-layer crashes in seconds.
3. **Run every script with a minimal VALID input**, not just `--help`, and record each exit code.
4. **Always add an edge-case pass:** 0-byte file, fake content (text in a `.png`), nonexistent path, 1×1 image. Exception handling is where scripts break. Right RC + raw Traceback is still a finding (unfriendly error), not a pass.
5. **Verify outputs two ways:** machine checks (dimensions, ink coverage, safe area, QR decodability) **and** a visual spot-check. Thumbnail everything into one contact sheet, then split it before sending to vision.
6. **Hunt warnings separately.** Run the build once and `grep -i 'warn\|error'` the whole log — `tail -N` silently hides warning blocks that sit above the summary.
7. **Report a graded per-item table with evidence.** "All passed" without the table is not an audit; every claim needs the command and its output.

Detail, runnable harness and checklist: `references/full-flow-audit.md`.

### Exit codes: never read them through a pipe

`cmd | tail -5; echo $?` reports **tail's** status, not `cmd`'s. This manufactures false passes and false "not a bug" conclusions. Use one of:

```bash
set -o pipefail                      # then $? reflects any failing stage
cmd > /tmp/o 2>&1; echo "RC=$?"      # preferred: no pipe at all
run() { local n="$1"; shift; "$@" >/tmp/o 2>&1; printf "  %-24s RC=%d\n" "$n" $?; }
```

Also: `VAR=xxx script.py` does **not** pass `VAR` into the script — the shell just executes the file (RC=126 `Permission denied`). Only a wrapper that itself reads `$VAR` (e.g. `./build.sh`) honours that prefix. Use the interpreter path, or `$VAR script.py`.

### 扫描器报出的问题数，先过正反例再上报

**一个自己写的检测器，在未校验前它输出的数字只能当假设。** 实测形态：扫技能库“坏引用”报出
70 处，逐条核下去 **0 处是真的** —— 扫描器把正文里任何长得像 `scripts/x.py` 的字符串都当成
「必须在技能目录下存在」，而它们其实是指向 `/path/to/.hermes/scripts/` 的系统脚本，甚至还有一段
**故意写错的教学示例**。结论已当着用户面发出后才被推翻。

上报前必做三个控制组：

| 控制组 | 做法 | 期望 |
|---|---|---|
| **正例** | 手工造一个真缺陷（真引用一个不存在的本地文件）| 必须被报出来 |
| **负例** | 手工造一个假缺陷（正文提及外部路径 / 代码块里的示例）| 必须**不**被报出来 |
| **零例** | 在已知干净的目录上跑 | 必须返回 0 |

任一组不符，先修扫描器再说结论。**“先报案数量再逐条核实”是错的顺序** —— 数量一旦说出口，
用户对整个审计的信任就押在了那个数字上，后面发现全是假阳性时，真缺陷也跟着不值钱了。

同一条适用于“凭记忆断言某个机制存在”：「系统有 X 保护所以任务会被跳过」这类断言，
**先 grep 源码拿到 `文件:行` 再当事实使用**。源码里搜不到的功能就是不存在的功能，
而基于一个不存在的机制定出的全局规则，会让每台机器永远背着它的维护成本。

## Delivering fix proposals (bugs the user will apply himself)

When the audit finds code bugs in a project the user maintains, the deliverable is **one revision-spec document** — not patch files, not direct edits to his repo.

- Four fields per item, every time: **位置 (file:line) · 现状代码 · 改为 (paste-ready) · 验证命令 (with the expected RC/output)**.
- Grade P0/P1/P2 and state which are must-fix for this version.
- Include a **"不建议改的" section** listing what looks like a bug but must not be touched (cross-platform font stacks, intentional exit codes, "slow but fine"). This is the anti-over-engineering guard, and the user reads it.
- Close with a **copy-pasteable acceptance checklist** (one command per fixed item, with the required RC) plus a CHANGELOG draft.
- **No local absolute paths, hostnames, machine IDs or credentials** in the document — it gets pasted into a public repo.

### Verify the fix before you recommend it

- Edit a **copy** (`cp -r repo /tmp/x`) and run before/after on the same input; report the observed difference (old Traceback vs new one-line message; old 98.5% vs new 23.4%).
- When the fix touches a computation, validate against **the whole fixture set**, not only the case that exposed it. Fixing one layout while breaking another is the default failure mode.
- **Search the same file for a helper that already does it right.** The recurring shape: the correct utility already exists, with a comment naming exactly this pitfall, while the broken code 25 lines away hardcodes the wrong thing. Reuse beats rewriting — every from-scratch reimplementation tried first was worse.
- Don't shrink sample sizes for speed without measuring: downscaling an image metric from 1400px to 700px flipped one layout from 60% to 99%.
- Distinguish severity honestly: a nonsense metric that is only *printed* is misleading information, not a functional failure. Say so, and let the user schedule it.

## Pitfalls

- Do not split every development workflow into a separate skill; use labeled subsections here.
- Preserve support files under `references/`, `templates/`, or `scripts/`.
- For debugging, understand the bug before fixing it.
- For code review, do not skip security and quality gates.
- For fork/rewrite, preserve legal and branding notes.
- Never conclude "not a bug" from output that came through a pipe — check the real exit status first.
- Never write a proposed fix into a spec document without running it; unverified advice costs the user a broken commit.
- A passing script is not a correct script: separately confirm it *fails loudly* on bad input.
- A scanner's finding count is not evidence until it passes a positive/negative/zero control — 70 reported bugs that are all false positives cost more trust than reporting 0.
- An asserted mechanism ("the system protects against X") is not a fact until you have `file:line` from the source. Rules built on imaginary mechanisms outlive the problem they were meant to solve.

## Verification checklist

- [ ] The task is mapped to the correct subsection.
- [ ] Reproduction or inspection happened before changes.
- [ ] Support files were rehomed without path breakage.
- [ ] Verification was run or explicitly noted as impossible.
- [ ] The final response states what changed and what remains risky.
