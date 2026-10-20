# wpship Privacy Policy

_Last updated: 20 October 2026_

wpship is an open-source tool that runs entirely on your own computer. The author of wpship operates no servers, accounts, analytics or telemetry, and **receives no data from you or your sites.**

## What wpship handles

- **Your site's files**: themes, plugins, mu-plugins and (with `--uploads`) media.
- **Your WordPress database** (only with `--db`). It may contain personal data such as your site users' and commenters' names and email addresses.
- **Connection settings**: server address, SSH user and port, folder paths and site URLs.

## Where it goes

Data moves only between **your computer** and **the server you configure**, over your own SSH connection. Nothing is sent to the wpship author or any third party. After a push, wpship requests your live site's homepage once to check that it loads.

## What is stored, where, and for how long

| Data | Location | Retention |
|---|---|---|
| Connection settings | `.wpship/config` on your computer (permissions 600, git-ignored) | Until you delete it |
| Live-site backups (database + wp-content) | `~/wpship-backups/<site>/` on your server | The newest 5 are kept (configurable via `KEEP_BACKUPS`); older ones are deleted automatically |
| Local database backups made during `pull --db` | `.wpship/local-backups/` on your computer | Until you delete them |

wpship never reads, stores or transmits SSH private keys or passwords; login is handled by your system's `ssh` program.

## Your responsibilities

If your WordPress site holds other people's personal data, you remain responsible for it under the laws that apply to you. wpship only copies it between machines you control.

## Contact

Questions or concerns: open an issue at https://github.com/zaidbin/wpship/issues
