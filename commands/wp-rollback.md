---
description: Restore the live site from a wpship backup
argument-hint: "[backup-id]"
---

Load the wpship skill. Run `"${CLAUDE_PLUGIN_ROOT}/bin/wpship" backups` and show me the list. Pick $ARGUMENTS (or the latest if empty), explain what will be restored, ask me to confirm, then run `"${CLAUDE_PLUGIN_ROOT}/bin/wpship" rollback <ID> --yes` and report whether the site responds.
