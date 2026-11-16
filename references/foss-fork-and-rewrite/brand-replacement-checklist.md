# Brand Replacement Checklist — example-vps Case Study

Verified: 2026-06-02, fork of KEJILION.SH → DUKANG-VPS

## Files That Must Be Replaced

### Core Scripts
- [ ] `dukang.sh` (root)
- [ ] `cn/dukang.sh`, `en/dukang.sh`, `jp/dukang.sh`, `kr/dukang.sh`, `tw/dukang.sh`, `ru/dukang.sh`, `ir/dukang.sh`
  - **Warning**: 7 files, often missed en/cn/jp/kr/tw — copy from original before replacing

### Config Files
- [ ] `cloudflare.conf` — email, tokens, domains
- [ ] `nginx.local`, `sshd.local` — references
- [ ] `custom_mysql_config*.cnf`, `www*.conf`, `optimized_php.ini`, `valkey.conf` — check for brand refs

### CI/CD
- [ ] `.github/workflows/translate.yml` — update `input_file = 'dukang.sh'` and output paths
- [ ] `translate.py` — update `input_file` and output paths

### Documentation
- [ ] `README.md` — logo URL, install command, links, star history
- [ ] `README.tw.md`, `README.ja.md`, `README.kr.md`, `README.ru.md`, `README.fa.md` — all language variants

## Verification Commands

```bash
# 1. Check for leftover brand references
grep -rn 'old-brand' . --include='*.sh' --include='*.conf' --include='*.md' --include='*.yml' --include='*.py' | grep -v '.git'

# 2. Syntax check ALL shell scripts
for f in $(find . -not -path './.git/*' -name '*.sh'); do bash -n "$f" 2>/dev/null || echo "FAIL: $f"; done

# 3. Count function parity with original
diff <(grep '^[a-z_]*() {' old.sh | sed 's/() {//' | sort) \
     <(grep '^[a-z_]*() {' new.sh | sed 's/() {//' | sort)
# Should show only: old_name → new_name

# 4. File count parity
find . -not -path './.git/*' -type f | wc -l
# Should match original (minus deleted files like logs)

# 5. Diff summary (non-brand changes)
diff -rq <(cd original && find . -not -path './.git/*' -type f | sort) \
         <(cd new && find . -not -path './.git/*' -type f | sort)
```

## Common Gotchas

1. **Multiple files in subdirectories** — en/cn/jp/kr/tw often have translated `.sh` files that were NOT copied to new repo. Must copy explicitly.
2. **Large files (>1MB) + patch tool** — `patch` with `replace_all` on 600KB+ shell scripts fails because old_string matches multiple times. Use `sed -i 's/old/new/g'` instead for bulk replacements.
3. **Patch tool line-ending mismatch** — `patch` sometimes fails on 400KB+ files with "content length differs" error. Re-read the file and retry, or fall back to `sed`.
4. **`.git-credentials` stale entries** — even after fixing `credential.helper`, old `ghp_...` tokens in `~/.git-credentials` can interfere.
5. **Logo/image URLs** — README images often point to external hosting. Replace with GitHub raw URLs or your own domain.
6. **Star History links** — `api.star-history.com/svg?repos=old/repo` must be updated to new owner/repo.
