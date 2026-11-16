# send_stats 替换调试记录

## Session: 2026-06-01 example-vps fork

### Problem
Replacing `send_stats()` function calls in a 22K-line bash script. Every sed/python approach broke bash syntax.

### What Failed (and Why)

#### 1. `sed 's/send_stats.*{/{# 统计已关闭/g'`
Destroyed the function definition. `{#` is not valid bash — bash only recognizes `#`.

#### 2. `sed -i '/^send_stats() {/,/^}/c\...'`
Same issue — produced `{#` artifacts. Also used sed multiline syntax with Chinese characters and escaped tabs, which produced broken output.

#### 3. Commenting out calls: `sed 's/send_stats ".*"/#send_stats/g'`
**Destroyed `;;` in case blocks.** When `; send_stats "xxx" ;` became `; #send_stats ;`, the `#` consumed everything after it including the closing `;`. Case blocks lost their terminators → "syntax error near unexpected token `;;'"`

#### 4. Python regex `; send_stats "xxx" ;` → `;`
Fixed the semicolon issue but didn't handle multi-line `if/then` blocks where a lone `#send_stats` comment left the then-branch empty → "syntax error near unexpected token `else'"

#### 5. `sed 's/# send_stats/#send_stats/g'`
Changed format but didn't address the root issue — `;;` separators were already being eaten.

### What Worked

**Disable the function definition itself.** Replace `send_stats() { ... }` with a body containing only `return`:

```python
content = re.sub(
    r'(send_stats\(\) \{)(.*?)(\n\})',
    r'\1\n\treturn\n\2',
    content, flags=re.DOTALL
)
```

This makes every `send_stats "xxx"` call instantly return — no syntax structure changes needed. **Always verify with `bash -n` afterward.**

### Key Lesson
For bash scripts, **modifying the function definition is safer than commenting out calls**. The latter destroys `;;` separators, `if/then` block emptiness, and multi-line structures that sed/python regex can't reliably handle in bulk.

### Verification
```bash
bash -n script.sh  # MUST pass after ANY replacement
# Also count ;; to verify: original 1004, should remain ~1004
grep -c ';;' script.sh
```
