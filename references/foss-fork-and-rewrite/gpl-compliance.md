# GPL-3.0 Compliance Quick Guide for Forks

## When Does This Apply?

When your fork's original project uses **GPL-3.0** or **GPL-2.0**.

## Requirements

1. **Keep the original license** — You must distribute your fork under the same GPL license. Do NOT change `LICENSE` to MIT, Apache, etc.
2. **Keep copyright notices** — Do NOT remove the original author's copyright line. You can ADD your own below it.
3. **Provide source code** — Anyone who gets your binary/script can request the source. Since this is a CLI script, the GitHub repo IS the source.
4. **Notice modifications** — You must add a prominent notice that you modified the original work.

## What You CAN Do

- ✅ Change branding, names, colors, menus
- ✅ Add new features
- ✅ Fix bugs
- ✅ Remove stats/analytics (this is just removing functionality, not changing license)
- ✅ Change the package name and binary name (e.g., `k` → `du`)
- ✅ Change installation paths
- ✅ Add your own README and documentation

## What You MUST NOT Do

- ❌ Change the license to something incompatible
- ❌ Remove the original copyright
- ❌ Claim you wrote the original code

## Example LICENSE Addition

```
Original copyright:
  Copyright (C) 2024 OriginalAuthor

  This program is free software: you can redistribute it and/or modify
  it under the terms of the GNU General Public License as published by
  the Free Software Foundation, either version 3 of the License, or
  (at your option) any later license.

Modified by: YourName (2026)
Changes: Brand rebranding, feature additions, stats removal.
```

## Resources

- https://www.gnu.org/licenses/gpl-faq.html#ModifyGPL
- https://opensource.org/licenses/gpl-3.0
- https://choosealicense.com/licenses/gpl-3.0/
