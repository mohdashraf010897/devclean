# JetBrains IDEs

platform: macos, linux, windows
detect: test -d ~/Library/Application\ Support/JetBrains || test -d ~/.config/JetBrains || test -d "$APPDATA/JetBrains" || test -d "$LOCALAPPDATA/JetBrains"

## Scan

macOS paths:
- `du -sh ~/Library/Caches/JetBrains` — all IDE caches (IntelliJ, WebStorm, Android Studio, etc.)
- `du -sh ~/Library/Application\ Support/JetBrains` — settings + state per IDE version
- `du -sh ~/Library/Logs/JetBrains` (if exists)
- List versions found: `ls ~/Library/Application\ Support/JetBrains/`

Linux paths:
- `du -sh ~/.cache/JetBrains`
- `du -sh ~/.config/JetBrains`

Windows paths (Git Bash):
- `du -sh "$LOCALAPPDATA/JetBrains"` — all IDE caches and indexes
- `du -sh "$APPDATA/JetBrains"` — settings + state per IDE version
- List versions found: `ls "$APPDATA/JetBrains/"` and `ls "$LOCALAPPDATA/JetBrains/"`

## Analysis

**Caches (`~/Library/Caches/JetBrains/` / `$LOCALAPPDATA/JetBrains/`):**
- Safe — index files, compiled AST caches, search indexes; IDE rebuilds on next open
- Typically 1–5 GB across multiple IDEs; grows continuously without cleanup

**Application Support / Settings (`~/Library/Application Support/JetBrains/` / `$APPDATA/JetBrains/`):**
- Review — contains IDE version-specific directories (e.g., `IntelliJIdea2024.1/`, `WebStorm2025.1/`)
- Each version directory has: settings (user's), plugins, logs, scratches
- Old version directories (for IDEs no longer installed or majorly upgraded) are orphaned → safe to delete
- Active version directories contain user settings → flag as Review, not Safe
- Identify orphaned by cross-referencing with installed apps:
  - macOS: `ls /Applications | grep -i jetbrains`
  - Windows: check Add/Remove Programs or `winget list | grep -i jetbrains`

## Cleanup

Safe   | Clear all JetBrains caches (macOS) | `rm -rf ~/Library/Caches/JetBrains`
Safe   | Clear JetBrains logs (macOS) | `rm -rf ~/Library/Logs/JetBrains`
Review | Delete orphaned old IDE version data (macOS) | `rm -rf ~/Library/Application\ Support/JetBrains/<OldVersion>`
Safe   | Clear all JetBrains caches (Linux) | `rm -rf ~/.cache/JetBrains`
Review | Delete orphaned old IDE version data (Linux) | `rm -rf ~/.config/JetBrains/<OldVersion>`
Safe   | Clear all JetBrains caches (Windows, Git Bash) | `rm -rf "$LOCALAPPDATA/JetBrains"`
Review | Delete orphaned old IDE version data (Windows, Git Bash) | `rm -rf "$APPDATA/JetBrains/<OldVersion>"`

## NEVER Delete

- `~/Library/Application Support/JetBrains/<ActiveVersion>/options/` — user settings (keymap, editor prefs, etc.)
- `~/Library/Application Support/JetBrains/<ActiveVersion>/scratches/` — scratch files (user-authored)
- `~/Library/Application Support/JetBrains/<ActiveVersion>/plugins/` — installed plugins
- `$APPDATA/JetBrains/<ActiveVersion>/options/` — user settings (Windows)
- `$APPDATA/JetBrains/<ActiveVersion>/scratches/` — scratch files (Windows)
- `$APPDATA/JetBrains/<ActiveVersion>/plugins/` — installed plugins (Windows)
