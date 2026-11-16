# Shell Script Rewriting Pitfalls — Session Evidence (2026-06-01)

## The example-vps Case Study

### Problem
Fork `kejilion/sh` → rebrand to `example-vps`. Need to:
1. Replace all brand strings
2. Disable telemetry (`send_stats`)
3. Verify `bash -n` passes after each step

### What Went Wrong

#### Mistake 1: `sed` for multi-line function replacement
```bash
# BROKEN: sed's c\ command fails on scripts with nested braces
sed -i '/^send_stats() {/,/^}/c\... ' dukang.sh
# Result: unmatched {' error, leaves `{# 统计已关闭...` garbage
```
**Lesson**: Never use `sed` for multi-line function rewrites on complex scripts.

#### Mistake 2: `re.sub` with `return` before original body
```python
# BROKEN: return exits the function, rest of body runs OUTSIDE
content = re.sub(
    r'(\1\n\treturn\n\2)',
    r'\1\n\treturn\n\2',
    content, flags=re.DOTALL
)
# Result: function loses closing }, syntax error at EOF
```
**Lesson**: Don't insert `return` into the middle of the original body. Replace the ENTIRE body with just `return`.

#### Mistake 3: Regex eats `;;` terminators in case branches
```bash
# BROKEN: sed pattern catches the ; before ;;
sed -i 's/send_stats "xxx" ;/; #send_stats/g'
# Result: case branches lose ;; → "syntax error near unexpected token `)'"
```
**Lesson**: Case branch patterns like `send_stats "xxx" ; cmd ;;` have fragile semicolons. Comment them out differently.

#### Mistake 4: `{#` is not a bash comment
```bash
# BROKEN: {# ... } looks like a comment but is syntax error
# The sed replacement left {# 统计已关闭docker_name}管理"
# Bash interprets { as a brace expansion, not comment
```
**Lesson**: Bash only supports `# ...` at line start. No `{# ... }` syntax.

### The Working Approach (v2)

1. **Copy original** → `dukang_new.sh = cp kejilion-original/kejilion.sh`
2. **Batch-replace brand strings** (duck → dukang, 科技lion → 嘟嘟)
3. **Replace `send_stats()` function** with:
   ```python
   content = content.replace(
       'send_stats() {\n\tlocal country=...',
       'send_stats() {\n\treturn\n}'
   )
   ```
   Only the closing `}` of the function — replace the ENTIRE body.
4. **Verify `bash -n`** → if pass, proceed. If fail, fix.
5. **Comment out calls** pattern-by-pattern using Python regex, verify after each batch.

### Verification Checklist
- [ ] `bash -n file.sh` passes
- [ ] `grep -c "KEJILION\|kejilion\|科技lion" file.sh` = 0 (except license comments)
- [ ] `grep -c "dukang\|嘟嘟" file.sh` > 0 (brand replaced)
- [ ] `grep -c "ENABLE_STATS" file.sh` shows `"false"`
- [ ] diff count vs original — understand what changed and why
