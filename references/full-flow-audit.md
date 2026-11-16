# Full-flow project audit — procedure and harness

For "run every flow and every step, find bugs I haven't found". The goal is a graded defect list backed by commands and output, delivered so the user can fix the code himself.

## Phase 0 — Build the checklist from the filesystem

Enumerate, do not recall:

```bash
find . -name '*.py' -not -path './.git/*' | sort        # scripts + tests
find . -name '*.md' -path './references/*' | sort        # docs
find . -name '*.sh' -o -name '*.ps1' | sort              # wrappers, and PLATFORM PAIRS
find . -path './example/*' -type f | sort                # fixtures
```

Cross-checks that are cheap and find real defects:

- **Orphan docs**: for each `references/*.md`, does any other doc mention its basename? An unreferenced doc is unreachable by on-demand loading — the capability exists but is invisible.
- **Platform twins**: if there is a `build.sh` and a `build.ps1`, diff their *supported option sets*, not their text. A fix applied to one and not the other leaves half the users broken.
- **Root-vs-package divergence**: a second copy of `SKILL.md`/`README` at repo root is frequently stale. Check whether anything links to it before deciding to delete vs resync.

## Phase 1 — Static gates (seconds, no fixtures needed)

```bash
for f in scripts/*.py; do python -m py_compile "$f" || echo "COMPILE FAIL: $f"; done
for f in scripts/*.py; do python "$f" --help >/dev/null 2>&1 || echo "HELP FAIL($?): $f"; done
```

`--help` failing usually means a crashing import or a malformed `add_argument` — both are real defects, cheap to find here.

## Phase 2 — Run every entry point

Use a harness that cannot lie about exit codes:

```bash
run() { local n="$1"; shift; "$@" >/tmp/o 2>&1; local rc=$?;
        printf "  %-28s RC=%-2d  %s\n" "$n" "$rc" "$(head -1 /tmp/o | cut -c1-52)"; }

run "build"      env PYTHON=python ./build.sh
run "doctor"     python scripts/doctor.py
run "pack(bad)"  python scripts/export_pack.py --root /tmp/nonexistent
```

Then assert the *expected* code for each: success paths RC=0, unknown-option paths RC=2, missing-input paths RC!=0 with a one-line message.

**Every subcommand, not just the top-level help.** Multi-command scripts hide divergence: two subcommands can take the same option name with two different formats, and only one of them validates.

## Phase 3 — Edge inputs (highest bug yield)

```bash
: > /tmp/e0.png                      # 0 bytes
echo "not an image" > /tmp/e1.png    # wrong content, right extension
python -c "from PIL import Image; Image.new('RGB',(1,1),'white').save('/tmp/e2.png')"
```

Run each consumer against all three plus a nonexistent path. Classify:

| Behaviour | Verdict |
|---|---|
| Friendly one-line error, nonzero RC | pass |
| Raw Traceback, nonzero RC | **finding** (unfriendly error) |
| RC=0 on bad input | **finding** (silent failure / false success) |

Also probe each option that accepts a structured string with a malformed value (`--region 100,100,300,200` where `name:` is required). Structured-option parsers are where crash-on-bad-input lives.

## Phase 4 — Output verification (two independent channels)

Machine checks are necessary but not sufficient — a layout can be pixel-perfect and still wrong.

**Contact sheet first, then split it:**

```python
from PIL import Image
cell = 460
sheet = Image.new('RGB', (cell*3, cell*3), '#e8e8e8')
# paste thumbnails, label each with draw.text
sheet.save('/tmp/sheet.png')
Image.open('/tmp/sheet.png').crop((0,0,W,H//2)).resize((1000,500)).save('/tmp/top.png')
```

Sending one big contact sheet to a vision model times out more often than sending two half-size crops. If a vision call times out on a size that succeeded moments earlier, **retry the identical image once before shrinking it** — the failure is intermittent, not dimensional. Do not re-render the artifact because of a vision timeout.

Ask the vision pass for *specific* defect classes only: blank output, text overlap/clipping/overflow, `□` tofu or missing glyphs, layout misplacement, outlier inconsistency. Open-ended "review this" invites confident filler.

## Phase 5 — Warning sweep

```bash
./build.sh 2>&1 | tee /tmp/build.log >/dev/null
search for 'warn|error|deprecat|unknown' in /tmp/build.log
```

Warnings that repeat once per render are usually a fallback chain doing its job (e.g. a cross-platform font stack warning about the absent platform's font name). **Verify harmlessness before reporting it as a bug**: check that glyphs actually rendered (ink coverage in a text region), then classify as noise and explicitly tell the user *not* to "fix" it by trimming the font stack — that would break the other platform.

## Deliverable shape

One markdown document. Per item:

```
### R<n>　<title>　[P0|P1|P2]
### 位置
scripts/foo.py:417  (+ every other occurrence — grep the pattern, not the line)
### 现状
<paste the offending code verbatim>
### 问题
<mechanism, plus the observable symptom the user would hit>
### 改为
<paste-ready replacement>
### 验证　✅实测
<commands + the exact expected output>
```

Then: graded summary table → "不建议改的" section → copy-pasteable acceptance checklist → CHANGELOG draft.

Keep the document free of local absolute paths, hostnames, machine IDs, tokens, and account names — it is destined for a public repository. Scan for them before delivering.

## Reporting style

- Lead with whether the happy path is healthy, then the defects. Do not bury a clean result under caveats.
- Give each defect a *consequence in the user's terms* ("the section name is wrong, so you think you built one and actually built nine"), not just the code shape.
- Admit gaps in your own method when you find them (e.g. "I truncated the build log and missed the warnings"). It is a finding about the audit, and it makes the rest of the report credible.
- Never soften a P0 into prose. Two must-fix items are two must-fix items.
