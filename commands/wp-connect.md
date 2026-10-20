---
description: Connect this local WordPress site to a VPS/host over SSH
argument-hint: "[host] [user] [live-url]"
---

Load the wpship skill and follow its rules. Set up `.wpship/config` for the WordPress site in the current folder using these details from the user: $ARGUMENTS. Ask only for what is missing (host, SSH user, port, key, WordPress folder on the server, live URL). Detect the local URL and local WP-CLI command yourself. Then run `"${CLAUDE_PLUGIN_ROOT}/bin/wpship" check` and fix or explain anything it reports.
