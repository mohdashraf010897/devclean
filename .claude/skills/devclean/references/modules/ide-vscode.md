# VS Code

platform: macos, linux, windows
detect: which code || test -d ~/Library/Application\ Support/Code || test -d ~/.config/Code || test -d "$APPDATA/Code"

## Scan

macOS paths:
- `du -sh ~/Library/Application\ Support/Code/CachedData`
- `du -sh ~/Library/Application\ Support/Code/Cache`
- `du -sh ~/Library/Application\ Support/Code/CachedExtensionVSIXs`
- `du -sh ~/Library/Application\ Support/Code/User/workspaceStorage`
- `du -sh ~/Library/Application\ Support/Code/logs`

Linux paths (use `~/.config/Code/` as base):
- Same structure under `~/.config/Code/`

Windows paths (Git Bash, base: `$APPDATA/Code/`):
- `du -sh "$APPDATA/Code/CachedData"`
- `du -sh "$APPDATA/Code/Cache"`
- `du -sh "$APPDATA/Code/CachedExtensionVSIXs"`
- `du -sh "$APPDATA/Code/User/workspaceStorage"`
- `du -sh "$APPDATA/Code/logs"`
- Note: VS Code Insiders uses `$APPDATA/Code - Insiders/` as the base path

## Analysis

All cache directories are Safe — VS Code regenerates them on next launch.
`workspaceStorage` is Safe — per-workspace metadata, not user-authored content.
`User/settings.json` and `User/keybindings.json` are user data — never touch.

Note: VS Code does not have a `state.vscdb.backup` issue like Cursor — its state DB is typically small.

## Cleanup

Safe | Clear extension + UI caches (macOS) | `rm -rf ~/Library/Application\ Support/Code/CachedData ~/Library/Application\ Support/Code/Cache ~/Library/Application\ Support/Code/CachedExtensionVSIXs`
Safe | Clear workspace storage (macOS) | `rm -rf ~/Library/Application\ Support/Code/User/workspaceStorage`
Safe | Clear logs (macOS) | `rm -rf ~/Library/Application\ Support/Code/logs`
Safe | Clear extension + UI caches (Linux) | `rm -rf ~/.config/Code/CachedData ~/.config/Code/Cache ~/.config/Code/CachedExtensionVSIXs`
Safe | Clear workspace storage (Linux) | `rm -rf ~/.config/Code/User/workspaceStorage`
Safe | Clear logs (Linux) | `rm -rf ~/.config/Code/logs`
Safe | Clear extension + UI caches (Windows, Git Bash) | `rm -rf "$APPDATA/Code/CachedData" "$APPDATA/Code/Cache" "$APPDATA/Code/CachedExtensionVSIXs"`
Safe | Clear workspace storage (Windows, Git Bash) | `rm -rf "$APPDATA/Code/User/workspaceStorage"`
Safe | Clear logs (Windows, Git Bash) | `rm -rf "$APPDATA/Code/logs"`

## NEVER Delete

- `~/Library/Application Support/Code/User/settings.json`
- `~/Library/Application Support/Code/User/keybindings.json`
- `~/Library/Application Support/Code/User/snippets/`
- `~/.vscode/extensions/` — installed extensions (takes time to reinstall)
- `$APPDATA/Code/User/settings.json` — user settings (Windows)
- `$APPDATA/Code/User/keybindings.json` — key bindings (Windows)
- `$USERPROFILE/.vscode/extensions/` — installed extensions (Windows; same warning applies)
