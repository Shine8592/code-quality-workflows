# Fork Comparison Template

Template for comparing file lists between upstream source and fork target during brand replacement forks.

## Usage

Run this comparison **before declaring the fork done**:

```bash
# Compare file lists
diff <(cd upstream && find . -not -path './.git/*' -type f | sed 's|^\./||' | sort) \
     <(cd fork && find . -not -path './.git/*' -type f | sed 's|^\./||' | sort)
```

## What to Look For

- Files in upstream but not in fork → **copy them** (especially `.sh` files in subdirectories)
- Files in fork but not in upstream → **investigate** (may be left-over old-name files)
- Same file count ≠ same files (this is the common trap)

## Real Example: KEJILION → DUKANG

```
Upstream (kejilion-original): 72 files
Fork before fix: 72 files ← same count, wrong files!
Fork after fix: 77 files ← +5 subdirectory .sh + CI workflow
```

The trap: a previous fork creator processed some subdirectories but not others, leaving `en/kejilion.sh`, `cn/kejilion.sh` etc. alongside missing `en/dukang.sh`, `cn/dukang.sh`.
