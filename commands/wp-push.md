---
description: Back up the live site, then push local changes to it
argument-hint: [--db] [--uploads] [--delete]
---

Load the wpship skill and follow its rules strictly. Run `"${CLAUDE_PLUGIN_ROOT}/bin/wpship" push $ARGUMENTS --dry-run`, explain the plan and changed files in plain words, and ask me to confirm. Only after I confirm, run it for real with `--yes`. Report the backup ID and how to roll back.
