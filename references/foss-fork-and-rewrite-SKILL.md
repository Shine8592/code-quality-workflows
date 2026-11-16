---
name: foss-fork-and-rewrite
description: "Fork and rebrand open-source CLI tools/scripts into your own project — license check, brand replacement, dependency cleanup, independent repo creation."
version: 1.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [OpenSource, FOSS, Fork, Rebrand, License, CLI, Shell]
    related_skills: [github-auth, github-repo-management, fork-brand-repair]
---

# FOSS Fork & Rewrite

Process for forking an open-source CLI tool/script and creating an independent derivative project with your own branding. This umbrella skill covers two major workflows:

1. **FOSS Fork & Rewrite** — License-check → brand replacement → dependency cleanup → independent repo creation. For smaller, single-file CLI tools.
2. **Fork Brand Repair** — Deep fork of large shell toolbox projects (10K+ lines) with per-language subdirectories, CI workflows, multi-README variants, and complex telemetry removal.

**Absorbed skills:** `fork-brand-repair` (merged into this umbrella).

Related skills: `github-auth`, `github-repo-management`.

## Prerequisites

1. **Check the license** — must be permissive (MIT, Apache-2.0, GPL, BSD, MPL).
   - GPL allows modification and redistribution but requires derivative works to remain GPL.
   - MIT/Apache: more flexible.
   - No license = cannot legally fork.
2. **Check attribution requirements** — some licenses require keeping copyright notices or adding your own.

## Workflow

### Phase 1: Clone and Analyze

```bash
git clone <repo-url> <project-name>-clone
cd <project-name>-clone
wc -l *.sh          # File sizes
find . -name "*.sh" | wc -l  # Total scripts
grep -r "brand\|author\|license" * | head -20  # Identify branding
```

### Phase 2: Brand Replacement

```bash
# Replace all instances of old brand with new brand
for f in *.sh; do
    sed -i 's/old-brand/new-brand/g' "$f"
    sed -i 's/OLD_BRAND/NEW_BRAND/g' "$f"
    sed -i 's/OldBrand/NewBrand/g' "$f"
done

# Handle subdirectories (e.g., localization folders)
for lang in cn en jp; do
    [ -d "$lang" ] && for f in "$lang"/*.sh; do
        sed -i 's/old-brand/new-brand/g' "$f"
    done
done

# Verify no leftovers
grep -r "old-brand" *.sh  # Should return nothing
grep -r "OLD_BRAND" *.sh  # Should return nothing
```

### Phase 3: Remove External Dependencies

Check for:
- **Stats/analytics endpoints** — `api.oldbrand.pro`, `events.oldbrand.com` — remove or comment out
- **Third-party URLs** — affiliate links, sponsor pages, donation addresses
- **External config** — `cloudflare.conf`, `nginx.local` with old-brand references

### Phase 4: Rewrite README

- Replace all brand names, logos, references
- Keep attribution to original work if required by license
- Add your own description, installation instructions, feature list

### Phase 5: Create Independent Repo

```bash
# Initialize new git repo
rm -rf .git
git init
git add -A
git commit -m "Initial commit: fork of <original-project> rebranded as <new-project>"

# Create new GitHub repo and push
gh repo create <new-project> --public --source=. --remote=origin
git push -u origin main
```

### Phase 6: Final Cleanup

- [ ] All brand references replaced
- [ ] Stats/analytics endpoints removed
- [ ] Affiliates/donation links updated or removed
- [ ] README fully rewritten
- [ ] License file preserved (if required)
- [ ] New git history started
- [ ] Tests pass (if applicable)

## Pitfalls

### Message Platform Filtering (QQ/Telegram/WeChat)

- **Shell variable `$` in loops**: Always use `$lang` not `lang` in shell `for` loops. Without `$`, the variable becomes a literal string and the directory lookup fails silently.
- **Path truncation**: Messaging platforms auto-filter strings matching `ghp_`, `gh_token`, `/path/to/.hermes/` and similar patterns, replacing them with `...`. Files written through the agent via messaging platform will contain truncated content. **Always create files on the VPS directly**, never transmit file paths through messages.
- **`bash << 'EOF'` heredocs**: Use quoted `'EOF'` (not `EOF`) so the shell doesn't try to expand variables before sending.
- **`sed` multi-line replacement**: `sed -i '/^send_stats() {/,/^}/c\...'` often fails with "unmatched {'" on platforms that truncate `\` or `'`. Use `python3 -c` for multi-line replacements, or use `grep` + comment-prefix approach for simpler cases.

### Shell Script Telemetry Removal

- **CRITICAL RULE**: When rewriting shell scripts, ALWAYS verify syntax after EVERY step with `bash -n file.sh`. Do NOT batch-replace all at once — do it incrementally and check each step.

- **`send_stats` pattern**: Shell scripts often have a stats/analytics function plus hundreds of calls.
  - **Correct approach**: Replace the function definition with just `send_stats() { return }` (single line), THEN comment out all calls individually.
  - **WRONG approach #1**: Using `re.sub(r'(\1\n\treturn\n\2')` on the function body — the `return` is inserted BEFORE the original body, so the rest of the body runs OUTSIDE the function (missing `}`), breaking structure.
  - **WRONG approach #2**: Using `sed -i 's/send_stats .*/#send_stats/g'` on case branches — this eats the `;;` terminator, causing "syntax error near unexpected token `)'" errors.
  - **WRONG approach #3**: Using `{# comment` — bash does NOT support `{# ... }` as a multi-line comment. It is a syntax error. Bash comments are always `# ...` (hash at line start).
  - **WRONG approach #4**: Using `sed '/^send_stats() {/,/^}/c\...'` — often fails with "unmatched {'" and is fragile.
  - **Best approach**: Copy the original file first, then use Python `re.sub` to replace the function body with ONLY `return`, verify with `bash -n`, then comment out calls one pattern at a time and verify.

- **Pattern-by-pattern comment-out**: Shell scripts have many `send_stats` call patterns:
  - `send_stats "action"` (standalone line)
  - `send_stats "action" ;` (followed by `;`)
  - `send_stats "action"; cmd` (on same line as command)
  - `1) send_stats "action" ; cmd` (case branch)
  - `if ...; then send_stats "action"; else ...` (if/then/else)
  - Each pattern needs different regex. Test after each batch.

- **Verify**: After removal, run `grep -c "send_stats" file.sh` — calls should be commented out (start with `#` or `:`). Check `ENABLE_STATS` is set to `"false"`.
- **Telemetry endpoints**: Also check for `api.*.pro`, affiliate URLs, sponsor links, and donation wallet addresses. Remove or replace with your own.

### gh Auth After VPS Reboot

- After VPS reboot, `gh` credential cache in `/root/.config/gh/hosts.yml` may be cleared. The `@reboot` cron auto-login script handles this (see `github-auth` skill).
- **Critical**: When `gh repo create` creates a repo but fails with "Unable to add remote", run `git remote remove origin` then `git remote add origin https://github.com/owner/repo.git` manually before push.

### patch Tool Failures on Large Shell Scripts

- **Symptom**: `patch` with `replace_all=True` fails with "Found N matches" (N > 1) on 600KB+ `.sh` files, because the old_string pattern appears in multiple places.
- **Fallback**: Use `sed -i 's/old/new/g' file` for bulk single-line replacements on large files. For multi-line changes, re-read the file first to get exact line numbers and context.
- **Line-ending mismatch**: `patch` sometimes reports "content length differs" on 400KB+ files. Re-read and retry, or fall back to `sed`.

### GPL Compliance

- If the original is GPL, your fork MUST remain GPL. Don't change the license.
- Most licenses require keeping the original copyright notice. Add your own below, not instead of.

## Git Credential Pitfalls

### `credential.helper = store` Breaks `gh` Auth (Dukang-vps 2026-06-02)

- **Symptom**: `git push` blocks repeatedly. The remote URL contains a hard-coded token (`https://ghp_...@github.com/...`) but the token in `~/.git-credentials` is stale/empty.
- **Root cause**: `credential.helper = store` takes priority over `!gh auth git-credential`. Git first tries the store's stale credentials, ignores the working gh token.
- **Fix**:
  ```bash
  # Step 1: Strip hard-coded tokens from ALL repo remote URLs
  git remote set-url origin https://github.com/owner/repo.git
  
  # Step 2: Replace credential.helper with ONLY gh
  git config --global --unset credential.helper
  git config --global credential.helper '!gh auth git-credential'
  
  # Step 3: Verify — should output only `!gh auth git-credential`
  git config --global --get credential.helper
  
  # Step 4: Push test
  git push --dry-run  # then git push
  ```
- **Also check**: `~/.git-credentials` may contain old `ghp_...` entries. These can also interfere — remove them when switching to gh-only auth.

## See Also

- `references/gpl-compliance.md` — Quick guide on GPL-3.0 compliance for forks
- `references/shell-rewriting-pitfalls.md` — Session evidence: why sed/re.sub broke shell scripts, the working v2 approach, `{#` is not a bash comment, `;;` terminator pitfall, example-vps case study
- `references/brand-replacement-checklist.md` — Step-by-step checklist for fork-and-rebrand, verification commands, common gotchas (large file patch failures, subdirectory translations missing, credential helper priority issue)

## See Also (absorbed from fork-brand-repair)

See `references/fork-example-vps-session-notes.md` — full session log of a real large-fork (KEJILION → DUKANG).
`references/send-stats-replacement-pitfalls.md` — why every sed/python approach to `send_stats` replacement fails and what works.
`templates/fork-file-comparison-template.md` — file-list diff template for verifying fork completeness.

---

# Fork Brand Repair — Deep Fork Workflow

Use this section for large shell-toolbox projects (10K+ lines, multi-language subdirectories, CI workflows). See references below for session-specific lessons.

### Phase 1: Clone and Analyze

```bash
# Clone the source
git clone https://github.com/upstream/repo.git repo-clone && cd repo-clone

# Find ALL brand traces (case-insensitive, exclude .git)
grep -ril "UPSTREAM_BRAND\|UPSTREAM_ALIAS\|UPSTREAM_PRO" . --exclude-dir=.git 2>/dev/null | sort
```

**Analyze the output before touching anything.** Check:
- Brand name variants (uppercase, camelCase, language variants)
- URL references (`UPSTREAM.sh`, `api.UPSTREAM.pro`, `github.com/UPSTREAM/repo`)
- Logo/image URLs
- Donation/tracking endpoints
- Copyright/license headers
- File/directory names referencing the original brand
- README and documentation content

### Phase 2: Brand Replacement

### Step 1: Replace all brand strings (simple)

```bash
for f in $(find . -not -path './.git/*' -type f \( -name "*.sh" -o -name "*.py" -o -name "*.yml" -o -name "*.md" -o -name "*.conf" -o -name "*.json" -o -name "*.txt" \) -print0 | xargs -0 grep -il "OLD_BRAND\|UPSTREAM_BRAND"); do
    sed -i 's|OLD_BRAND|NEW_BRAND|g' "$f"
done
```

Run multiple passes for different formats:
- `UPSTREAM` → `NEW_BRAND` (full word)
- `upstream` → `new_brand` (lowercase)
- `Upstream` → `New Brand` (camelCase)

### Step 2: Python for complex multi-pass replacement

See `references/shell-rewriting-pitfalls.md` for why sed-based approaches break on complex scripts.

### Step 3: Rename files and directories referencing the brand

```bash
mv ORIGINAL_NAME.sh NEW_NAME.sh
mv brand-dir/ newbrand-dir/
```

### Phase 2.5: Bash Syntax Verification (CRITICAL)

**每次品牌替换后必须验证 bash 语法，尤其在注释掉 `send_stats()` 函数及其调用时。**

```bash
bash -n dukang.sh
```

### send_stats handling — the RIGHT way

**BAD:** Use sed to comment out `send_stats` calls — destroys `;;` in case blocks:
```bash
# BAD: destroys ;; in case blocks
sed -i 's/send_stats \".*/#send_stats/g' script.sh
```

**GOOD:** Disable the function definition itself (make it a no-op), keep ALL calls intact:
```python
content = re.sub(
    r'(send_stats\\(\\) \\{)(.*?)(\\n\\})',
    r'\\1\\n\\treturn\\n\\2',
    content, flags=re.DOTALL
)
```
This preserves `;;` case block terminators and `if/then/else/fi` structure.

### send_stats handling — if you MUST comment out calls

- `; send_stats "xxx" ;` → `: ;` (keep the ;; that follows on next line)
- Empty `if/then` branches → add `: ;` as placeholder
- `case N) ; #send_stats "xxx" ; action ;;` → `case N) ; : ; action ;;`

**Always run `bash -n` after changes. If it fails, the replacement strategy was wrong.**
See `references/send-stats-replacement-pitfalls.md` for the full debug transcript.

### Phase 3: Remove Tracking and Donations

```bash
# Remove all analytics/tracking API calls
grep -n "api\\.UPSTREAM\\|analytics\\|tracking\\|send_stats\\|telemetry" *.sh
# Comment out or remove the entire tracking function
```

### Donation links
```bash
grep -rn "TRC20\\|Bitcoin\\|Donate\\|donation\\|打赏\\|捐赠" *.md *.sh
```

### Phase 4: Final Verification

```bash
# Comprehensive check — ZERO residual
grep -ril "OLD_BRAND\\|UPSTREAM" . --exclude-dir=.git 2>/dev/null

# File count comparison (same count ≠ same files!)
diff <(cd upstream && find . -not -path './.git/*' -type f | sed 's|^\\./||' | sort) \\\
     <(cd fork && find . -not -path './.git/*' -type f | sed 's|^\\./||' | sort)

# Function list diff (function parity, not just file count)
grep '^[a-z_]*() {' upstream/script.sh | sed 's/() {//' | sort > /tmp/orig_funcs.txt
grep '^[a-z_]*() {' fork/script.sh | sed 's/() {//' | sort > /tmp/fork_funcs.txt
comm -23 /tmp/orig_funcs.txt /tmp/fork_funcs.txt  # missing functions
comm -13 /tmp/orig_funcs.txt /tmp/fork_funcs.txt  # unexpected additions
```

### README links pointing to external domains
When the original uses custom domains (e.g., `kejilion.sh`, `嘟嘟.sh`) for logo images, screenshots, and install URLs, these domains may not exist for your fork. **Replace with GitHub raw URLs as a temporary placeholder.**

### Verify all sub-directory scripts exist
Shell toolboxes often have translated copies in subdirectories (`cn/`, `en/`, `jp/`, `kr/`, `ru/`, `tw/`, `ir/`). A previous fork creator may only have processed some subdirs. **Always compare file lists between source and target BEFORE declaring done.**

### Zero residual verification is mandatory
After all replacements, run `grep -ril "OLD_BRAND" . --exclude-dir=.git`. If ANY files remain, the fork is NOT clean.

### Function list diff confirms 100% parity
File count matching is NOT sufficient proof of a complete fork. Only name differences (`kejilion()` → `dukang()`) are expected. Any missing functions means the fork is incomplete.

### Brand replacement in CI workflow
The `.github/workflows/translate.yml` often contains inline Python with hardcoded script filenames. Replace ALL occurrences.

### Git history retains old brand names
After creating the new repo with the first commit, the old brand names are only in the upstream repo's history. Use `git checkout --orphan` for a truly clean start if needed.

### Commenting out send_stats destroys ;; separators
When you replace `send_stats "xxx"` with `#send_stats`, the `;;` that follows on the **next line** remains — but if on the **same line** (e.g., `; send_stats "xxx" ;`), your sed pattern eats the closing `;`, breaking the case block.

### Same file count ≠ same files (the "incomplete fork" trap)
A previous fork creator may have processed SOME subdirectories but skipped others. **ALWAYS do a file list diff before declaring done:**
```bash
diff <(cd upstream && find . -not -path './.git/*' -type f | sed 's|^\\./||' | sort) \\\
     <(cd fork && find . -not -path './.git/*' -type f | sed 's|^\\./||' | sort)
```

### awk scripts fail bash -n (expected, not a failure)
Some `.sh` files are actually awk scripts with `#!/usr/bin/awk -f`. Skip bash syntax check for those.

### SSH key not set up, token-based git push hangs
```bash
git checkout --orphan gh-pages
git rm -rf . 2>/dev/null || true
# create index.html
git add -A && git commit -m "feat: publish site" && git push -f origin gh-pages
```

## Pitfalls

### User expects agent to execute commands, not ask user to run them
When the user says "嘟嘟，你可以自己执行的" — always execute commands directly. Only ask when a decision requires human judgment (e.g., "which domain to buy"), not for technical execution.

### .git directory always shows matches
`grep -r` will find brand names in `.git/objects/pack/*.pack`. Always exclude `--exclude-dir=.git`.

### Some files contain translated/localized text with brand names
Multi-language versions and translation helper scripts often contain brand references. Run replacements on ALL language directories.

### README.md text differs from README.md sed patterns
Use Python for comprehensive README cleanup.

### LICENSE files
**Do NOT modify the LICENSE** if it's a copyleft license (GPL, AGPL, MIT) — the original copyright stays. See `references/gpl-compliance.md` for full GPL compliance guide.

### .github/workflows referencing the old brand
CI/CD pipeline files may reference the old repo URL or script name. Always check and fix.

### Test files and fixtures
Replace in `tests/`, `cn/tests/`, etc.

### Large file patch failures
`patch` with `replace_all=True` fails with "Found N matches" on 600KB+ `.sh` files. Fallback: use `sed -i 's/old/new/g' file` for bulk single-line replacements. For multi-line changes, fall back to `sed`.

### `gh` auth after VPS reboot
After VPS reboot, `gh` credential cache in `/root/.config/gh/hosts.yml` may be cleared. See `github-auth` skill for auto-login setup.

### `credential.helper = store` Breaks `gh` Auth
When `git push` blocks repeatedly: strip hardcoded tokens from remote URLs, set `credential.helper` to `!gh auth git-credential` only, remove stale entries from `~/.git-credentials`.
```bash
git remote set-url origin https://github.com/owner/repo.git
git config --global --unset credential.helper
git config --global credential.helper '!gh auth git-credential'
```
