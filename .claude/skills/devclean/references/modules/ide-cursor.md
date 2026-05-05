# Cursor IDE

platform: macos, linux, windows
detect: test -d ~/Library/Application\ Support/Cursor || test -d ~/.config/Cursor || test -d "$APPDATA/Cursor"

## Scan

macOS paths:
- `du -sh ~/Library/Application\ Support/Cursor/User/globalStorage/state.vscdb.backup` (if exists)
- `du -sh ~/Library/Application\ Support/Cursor/User/globalStorage/state.vscdb` (size indicator only — NOT a cleanup candidate)
- `du -sh ~/Library/Application\ Support/Cursor/CachedData`
- `du -sh ~/Library/Application\ Support/Cursor/Cache`
- `du -sh ~/Library/Application\ Support/Cursor/CachedExtensionVSIXs`
- `du -sh ~/Library/Application\ Support/Cursor/User/workspaceStorage`
- `du -sh ~/Library/Application\ Support/Cursor/logs`

Linux paths (substitute `~/.config/Cursor/` for `~/Library/Application Support/Cursor/`):
- Same structure, different base path

Windows paths (Git Bash, base: `$APPDATA/Cursor/`):
- `du -sh "$APPDATA/Cursor/User/globalStorage/state.vscdb.backup"` (if exists) — backup DB
- `du -sh "$APPDATA/Cursor/User/globalStorage/state.vscdb"` (size indicator only — NOT a cleanup candidate)
- `du -sh "$APPDATA/Cursor/CachedData"`
- `du -sh "$APPDATA/Cursor/Cache"`
- `du -sh "$APPDATA/Cursor/CachedExtensionVSIXs"`
- `du -sh "$APPDATA/Cursor/User/workspaceStorage"`
- `du -sh "$APPDATA/Cursor/logs"`

## Analysis

**state.vscdb.backup:**
- Safe — explicit backup file; Cursor recreates it automatically
- Can grow to 20+ GB; this is the #1 Cursor space hog

**state.vscdb (live DB):**
- Risky — the live global state database for all Cursor extensions
- Deleting it resets all extension state (settings, history, etc.)
- If abnormally large (> 500 MB), flag it as noteworthy but DO NOT offer deletion
- Suggest: investigate which extension is bloating it (usually GitLens or Java extension)

**CachedData, Cache, CachedExtensionVSIXs:**
- Safe — VSIX extension downloads and UI caches; rebuilt on next Cursor launch

**workspaceStorage:**
- Safe — per-workspace state (recently opened files, panel state); rebuilds automatically

**logs:**
- Safe — application logs; accumulate silently over time

## Cleanup

Safe   | Delete backup DB (macOS) | `rm -f ~/Library/Application\ Support/Cursor/User/globalStorage/state.vscdb.backup`
Safe   | Clear extension caches (macOS) | `rm -rf ~/Library/Application\ Support/Cursor/CachedData ~/Library/Application\ Support/Cursor/Cache ~/Library/Application\ Support/Cursor/CachedExtensionVSIXs`
Safe   | Clear workspace storage (macOS) | `rm -rf ~/Library/Application\ Support/Cursor/User/workspaceStorage`
Safe   | Clear logs (macOS) | `rm -rf ~/Library/Application\ Support/Cursor/logs`
Safe   | Delete backup DB (Linux) | `rm -f ~/.config/Cursor/User/globalStorage/state.vscdb.backup`
Safe   | Clear extension caches (Linux) | `rm -rf ~/.config/Cursor/CachedData ~/.config/Cursor/Cache ~/.config/Cursor/CachedExtensionVSIXs`
Safe   | Clear workspace storage (Linux) | `rm -rf ~/.config/Cursor/User/workspaceStorage`
Safe   | Clear logs (Linux) | `rm -rf ~/.config/Cursor/logs`
Safe   | Delete backup DB (Windows, Git Bash) | `rm -f "$APPDATA/Cursor/User/globalStorage/state.vscdb.backup"`
Safe   | Clear extension caches (Windows, Git Bash) | `rm -rf "$APPDATA/Cursor/CachedData" "$APPDATA/Cursor/Cache" "$APPDATA/Cursor/CachedExtensionVSIXs"`
Safe   | Clear workspace storage (Windows, Git Bash) | `rm -rf "$APPDATA/Cursor/User/workspaceStorage"`
Safe   | Clear logs (Windows, Git Bash) | `rm -rf "$APPDATA/Cursor/logs"`

## NEVER Delete

- `~/Library/Application Support/Cursor/User/globalStorage/state.vscdb` — live extension state database
- `~/Library/Application Support/Cursor/User/settings.json` — user settings
- `~/Library/Application Support/Cursor/User/keybindings.json` — key bindings
- `~/Library/Application Support/Cursor/User/snippets/` — user snippets
- `$APPDATA/Cursor/User/globalStorage/state.vscdb` — live extension state database (Windows)
- `$APPDATA/Cursor/User/settings.json` — user settings (Windows)
- `$APPDATA/Cursor/User/keybindings.json` — key bindings (Windows)
