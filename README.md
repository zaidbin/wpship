# wpship

**Ship a local WordPress site to *any* VPS or host over SSH — with automatic backups, dry runs and one-command rollback.**

Tools like WordPress Studio make building a site locally (with AI) easy, but they publish only to their own hosting. `wpship` is the missing last step: your DigitalOcean droplet, Hetzner box, Lightsail, or any server you can SSH into.

It works two ways:

- **As a Claude Code plugin** — say *"push my theme changes live"* and Claude dry-runs, shows you what will change, asks, then deploys.
- **As a plain CLI** — `wpship push`, no AI needed.

```
local site (Studio / LocalWP / any)  ──wpship push──▶  your server
        ▲                                                 │
        └──────────────── wpship pull ────────────────────┘
                 every push = automatic backup first
```

## Safety model

| Default | Why |
|---|---|
| Live site is **backed up before every push** (DB + wp-content) | `wpship rollback` undoes any push |
| Pushes **code only** (themes, plugins, mu-plugins) | Overwriting a live database wipes new comments, orders and users |
| `--db` makes you **type the domain** to confirm | No accidental database overwrites |
| Never deletes server files unless you pass `--delete` | Old plugins on the server stay put |
| Studio's SQLite files are never uploaded | They'd break a MySQL server |
| Refuses a DB push if table prefixes differ | Avoids a half-broken site |
| Checks the live site returns HTTP 200 after push | Tells you to roll back if not |
| SSH keys only, no passwords stored | Config never holds secrets |

## Requirements

- **Your computer:** bash, `ssh`, `tar`, and WP-CLI for your local site (`studio wp` works for WordPress Studio). macOS and Linux work as-is. On **Windows**, use **Git Bash** (comes with [Git for Windows](https://git-scm.com/download/win)); WSL works too.
- **Your server:** SSH key login and WordPress already installed. WP-CLI is optional — `wpship install-wpcli` adds it.
- **Optional: `rsync`** on both ends. With it, pushes send only changed files and `--delete` works. Without it, wpship streams whole folders with `tar` over SSH, which is slower but needs nothing extra installed. Force a mode with `TRANSFER=rsync` or `TRANSFER=tar` in the config.

### Windows (Git Bash)

```sh
git clone https://github.com/zaidbin/wpship ~/wpship
mkdir -p ~/bin && ln -s ~/wpship/bin/wpship ~/bin/wpship   # Git Bash has ~/bin on PATH
cd /c/Users/YOUR_NAME/Studio/my-site
wpship init
```

## Install

**As a Claude Code plugin**

```
/plugin marketplace add zaidbin/wpship
/plugin install wpship@wpship
```

**As a CLI only**

```sh
git clone https://github.com/zaidbin/wpship
ln -s "$PWD/wpship/bin/wpship" /usr/local/bin/wpship
```

## Quick start

```sh
cd ~/Studio/my-site          # your local WordPress folder
wpship init                  # server IP, SSH user, live URL…
wpship check                 # tests everything
wpship status                # what differs from live?
wpship push --dry-run        # see exactly what would happen
wpship push                  # backup → upload → verify
```

First deploy to a fresh server usually needs the database and media too:

```sh
wpship push --db --uploads
```

Something broke?

```sh
wpship backups               # list backups on the server
wpship rollback              # restore the latest one
```

### In Claude Code

| Slash command | Does |
|---|---|
| `/wp-connect` | Sets up the server connection and runs the checks |
| `/wp-status` | Shows what differs from live |
| `/wp-push [--db] [--uploads]` | Dry run → your OK → deploy |
| `/wp-pull [--db] [--uploads]` | Bring the live site down to local |
| `/wp-rollback [id]` | Restore a backup |

Or just talk: *"I added a plugin locally, ship it to the droplet."*

## Commands

```
init            Save server details (.wpship/config)
check           Test SSH, transfer mode, WP-CLI, remote WordPress
install-wpcli   Install WP-CLI on the server
status          Files that differ (local vs live)
push            Backup, then upload themes/plugins/mu-plugins
pull            Download from live
backup          Back up live now
backups         List server backups
rollback [ID]   Restore a backup (default: latest)

--db  --uploads  --delete  --dry-run  --yes  --path DIR
```

## Config

`.wpship/config` lives in your site folder, is `chmod 600`, and is git-ignored automatically:

```sh
SSH_HOST=203.0.113.10
SSH_USER=root
SSH_PORT=22
SSH_KEY=~/.ssh/id_ed25519
REMOTE_PATH=/var/www/html
REMOTE_URL=https://example.com
LOCAL_URL=http://localhost:8881
LOCAL_WP="studio wp"
REMOTE_WP=wp
REMOTE_OWNER=www-data:www-data   # chown after push; blank to skip
SYNC_DIRS="themes plugins mu-plugins"
KEEP_BACKUPS=5
```

Backups are stored on the server in `~/wpship-backups/<site>/`, outside the web root.

## Not supported yet

- Shared hosting without SSH (FTP/cPanel-only)
- Merging databases — `--db` replaces the live database entirely
- Multisite

PRs welcome.

## License

MIT
