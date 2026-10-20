---
name: wpship
description: Deploy, sync, back up or roll back a local WordPress site (WordPress Studio, LocalWP, wp-env or any folder with wp-config.php) to the user's own VPS or host over SSH. Use when the user wants to push changes live, pull the live site down, connect a server, check what differs, or undo a bad deploy.
---

# wpship — local WordPress → any SSH server

The `wpship` CLI lives at `${CLAUDE_PLUGIN_ROOT}/bin/wpship`. Always run it
from the local site folder (the one containing `wp-config.php`), or pass
`--path <folder>`. Call it as `W="${CLAUDE_PLUGIN_ROOT}/bin/wpship"` then `"$W" <command>`.

## Commands

| Command | What it does |
|---|---|
| `init` | Interactive: saves server details to `.wpship/config` (no passwords) |
| `check` | Verifies SSH key login, transfer mode (rsync or tar), WP-CLI, remote WordPress, siteurl |
| `install-wpcli` | Prints the official WP-CLI install steps for the user to run on the server |
| `status` | Lists theme/plugin files that differ (local vs live) |
| `push [--db] [--uploads] [--delete]` | Backs up live, then uploads |
| `pull [--db] [--uploads]` | Brings live files (and optionally DB) down |
| `backup`, `backups`, `rollback [ID]` | Safety net on the server |

Global flags: `--dry-run`, `--yes`, `--path DIR`.

## Rules you must follow

1. **Never run `push`, `pull` or `rollback` for real without the user's explicit OK in chat for that specific run.** First run it with `--dry-run`, show the user the plan and the file list in plain words, then ask.
2. **Default to a code-only push** (themes, plugins, mu-plugins). Only add `--db` when the user explicitly asks to push the database. Before a DB push, tell them plainly: live comments, orders, users and posts created since their last pull will be replaced, and live logins become their local users. If they agree, pass `--yes` (the tool would otherwise ask them to type the domain, which you can't do interactively).
3. Only add `--delete` when the user asks to remove files from the server.
4. **Never put passwords or private keys in chat, commands or the config.** `wpship` uses SSH keys. If key login fails, tell the user to set up SSH key login for that server themselves (their host's docs explain it). Never read, copy or print anything from the user's SSH folder.
5. `init` is interactive. Since you can't type into prompts, write `.wpship/config` directly instead, using values the user gives you (format below), then `chmod 600` it and create `.wpship/.gitignore` containing `*`.
6. Run `check` after setup and after any config change; fix what it reports before pushing.
7. After every real push, report the backup ID the tool printed and tell the user that `wpship rollback <ID>` undoes it. If the post-push HTTP check fails, offer the rollback straight away.

## Config file (`.wpship/config`, shell syntax)

```sh
SSH_HOST=203.0.113.10
SSH_USER=root
SSH_PORT=22
SSH_KEY=                           # optional key file path; blank = default key
REMOTE_PATH=/var/www/html          # folder with wp-config.php on server
REMOTE_URL=https://example.com
LOCAL_URL=http://localhost:8881    # get with: studio wp option get siteurl
LOCAL_WP="studio wp"               # "wp" for LocalWP / normal installs
REMOTE_WP=wp                       # WP-CLI command on the server
REMOTE_OWNER=www-data:www-data     # chown after push; blank = skip
SYNC_DIRS="themes plugins mu-plugins"
KEEP_BACKUPS=5
```

## Typical flows

- **First deploy to a new droplet:** user installs WordPress on the server (any method), you write the config, run `check`, `push --dry-run`, show plan, then after OK `push --db --uploads --yes` (first deploy usually needs the DB and media).
- **Everyday change** (edited a theme or generated a plugin locally): `status` → `push --dry-run` → OK → `push --yes`.
- **Start working from real data:** `pull --db --uploads --dry-run` → OK → `pull --db --uploads --yes`.
- **Something broke:** `backups` → `rollback <ID> --yes` after OK.

## Known limits (say so if relevant)

- Needs SSH key access. Shared hosting without SSH is not supported yet. On Windows it runs in Git Bash (or WSL); Studio sites live at `/c/Users/NAME/Studio/SITE`, and Studio's CLI may be `studio.cmd wp`.
- Without rsync on both ends, wpship uses tar over SSH: pushes send whole folders and `--delete` is refused. Suggest `sudo apt install rsync` on the server if pushes are slow.
- Local and live table prefixes must match for `--db` (the tool refuses otherwise).
- Studio sites use SQLite locally; DB export/import tries `wp db` then `wp sqlite`. If both fail, tell the user and push code only.
