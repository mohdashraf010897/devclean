# DevClean — Universal Safety Reference

## NEVER Delete — Cross-Platform

These paths must never be offered for deletion regardless of size.
Modules may add their own tool-specific entries.

### Credentials & Keys
- `~/.ssh/` — SSH keys
- `~/.gnupg/`, `~/.gpg/` — GPG keys
- `~/.aws/` — AWS credentials
- `~/.kube/` — Kubernetes configs
- `~/.docker/config.json` — Docker registry credentials
- `~/.android/adbkey`, `~/.android/adbkey.pub` — ADB auth keys
- `~/.netrc` — network credentials
- `~/.config/gh/` — GitHub CLI auth
- Any `credentials.json`, `*.pem`, `*.key`, `*.p12`, `*.pfx` files

### User Data
- `~/Documents/`, `~/Desktop/`, `~/Pictures/`, `~/Movies/` — user files
- `~/.gitconfig` — git identity and settings
- `~/.zshrc`, `~/.bashrc`, `~/.profile`, `~/.bash_profile` — shell config
- `~/.config/` — broad application config (never delete wholesale)
- Any `.env`, `.env.local`, `.env.production` files anywhere

### macOS-Specific
- `~/Library/Keychains/` — macOS keychain
- `~/Library/Application Support/Google/Chrome/Default/` — Chrome profile
- `~/Library/Application Support/Firefox/Profiles/` — Firefox profile
- `~/Library/Application Support/Cursor/User/globalStorage/state.vscdb` — live IDE DB (backup is safe, live file is NOT)
- `~/Library/Application Support/Notion/` — Notion local data

### Linux-Specific
- `~/.local/share/keyrings/` — GNOME keyring
- `/etc/` — system config
- `/var/lib/` — application data (databases, etc.)

### Windows-Specific
- `%APPDATA%\Microsoft\Credentials\` — Windows credentials
- `%USERPROFILE%\.ssh\` — SSH keys
- `$APPDATA/Code/User/settings.json`, `$APPDATA/Code/User/keybindings.json` — VS Code user settings
- `$APPDATA/Cursor/User/settings.json`, `$APPDATA/Cursor/User/keybindings.json` — Cursor user settings
- `$APPDATA/JetBrains/<ActiveVersion>/options/` — JetBrains user settings
- `$USERPROFILE/Documents/WindowsPowerShell/Microsoft.PowerShell_profile.ps1` — PowerShell profile
- `$USERPROFILE/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine/ConsoleHost_history.txt` — PowerShell command history
- `$USERPROFILE/scoop/persist/` — Scoop app persistent data (user configs for Scoop-managed apps)

---

## Risk Classification Definitions

These definitions are canonical. All modules must use these exactly.

**Safe** — Automatically rebuilt on next use. No user data. No rebuild cost
beyond time (e.g., re-downloading a package cache). Examples: Yarn cache,
Homebrew download cache, Xcode DerivedData, browser caches.

**Review** — May contain duplicates, orphans, or accumulated state that could
be cleaned — but requires semantic analysis first. Examples: Android AVDs
(read config.ini before deciding), iOS simulators (check version duplicates),
Xcode Archives (check dates before deleting old builds).

**Risky** — Contains user data, credentials, or live application state that
cannot be automatically recovered. Flag for awareness only. Never offer
deletion. Examples: browser profiles, IDE state databases, cloud CLI configs.

---

## Module Format Contract

Every module in `modules/` must follow this structure for the orchestrator to
read it correctly:

```
# Module Name

platform: macos|linux|windows|all   (comma-separate for multi-platform)
detect: <single shell command that exits 0 if toolchain is present>

## Scan
<du / find / tool-specific commands to measure what's installed>

## Analysis
<how to classify findings into Safe / Review / Risky>
<include any config-reading logic for semantic analysis>

## Cleanup
<exact shell commands for each cleanup action>
<pair each command with its Risk level>

## NEVER Delete
<tool-specific additions to the universal list>
```

---

## Platform Path Reference

Common path equivalents across platforms:

| Purpose | macOS | Linux | Windows |
|---------|-------|-------|---------|
| User home | `~/` | `~/` | `%USERPROFILE%\` |
| User cache | `~/Library/Caches/` | `~/.cache/` | `%LOCALAPPDATA%\` |
| App support | `~/Library/Application Support/` | `~/.local/share/` | `%APPDATA%\` |
| App logs | `~/Library/Logs/` | `~/.local/share/logs/` or `/var/log/` | `%LOCALAPPDATA%\` |
| Temp | `/tmp/` | `/tmp/` | `%TEMP%\` |

---

## Adding a Module

1. Copy `modules/TEMPLATE.md` to `modules/<toolname>.md`
2. Fill in all sections
3. Test the detect command on a clean machine
4. Submit a PR — the orchestrator picks it up automatically
