# Browser Caches

platform: macos, linux, windows
detect: test -d ~/Library/Caches/Google || test -d ~/Library/Caches/Firefox || test -d ~/.cache/google-chrome || test -d ~/.cache/mozilla || test -d "$LOCALAPPDATA/Google/Chrome"

## Scan

macOS:
- `du -sh ~/Library/Caches/Google` — Chrome/Chromium cache
- `du -sh ~/Library/Caches/Firefox`
- `du -sh ~/Library/Caches/Mozilla`
- `du -sh ~/Library/Caches/com.google.Chrome` (if exists)
- `du -sh ~/Library/Application\ Support/Google/Chrome/Default` — size indicator only (profile, NOT cache)

Linux:
- `du -sh ~/.cache/google-chrome`
- `du -sh ~/.cache/chromium`
- `du -sh ~/.cache/mozilla`

Windows (Git Bash):
- `du -sh "$LOCALAPPDATA/Google/Chrome/User Data/Default/Cache"` — Chrome cache only
- `find "$LOCALAPPDATA/Mozilla/Firefox/Profiles" -maxdepth 2 -name "cache2" -type d 2>/dev/null | xargs du -sh` — Firefox cache dirs only (NOT profiles)
- `du -sh "$LOCALAPPDATA/Microsoft/Edge/User Data/Default/Cache"` — Edge cache only
- Note: `$LOCALAPPDATA/Google/Chrome/User Data/Default/` is the Chrome profile (NOT cache) — flag size only, never offer to delete

## Analysis

**Cache directories (`~/Library/Caches/Google`, `~/.cache/google-chrome`, etc.):**
- Safe — HTTP cache, rendered page data, shader caches; all rebuild automatically

**Profile directory (`~/Library/Application Support/Google/Chrome/Default/`):**
- Risky — contains bookmarks, cookies, saved passwords, extensions, browsing history
- Flag size for awareness only; NEVER offer to delete
- Same for Firefox: `~/Library/Application Support/Firefox/Profiles/`

## Cleanup

Safe | Clear Chrome cache (macOS) | `rm -rf ~/Library/Caches/Google`
Safe | Clear Firefox cache (macOS) | `rm -rf ~/Library/Caches/Firefox ~/Library/Caches/Mozilla`
Safe | Clear Chrome cache (Linux) | `rm -rf ~/.cache/google-chrome`
Safe | Clear Firefox cache (Linux) | `rm -rf ~/.cache/mozilla`
Safe | Clear Chrome cache (Windows, Git Bash) | `rm -rf "$LOCALAPPDATA/Google/Chrome/User Data/Default/Cache"`
Safe | Clear Firefox cache only (Windows, Git Bash) | `find "$LOCALAPPDATA/Mozilla/Firefox/Profiles" -maxdepth 2 -name "cache2" -type d -exec rm -rf {} + 2>/dev/null`
Safe | Clear Edge cache (Windows, Git Bash) | `rm -rf "$LOCALAPPDATA/Microsoft/Edge/User Data/Default/Cache"`

## NEVER Delete

- `~/Library/Application Support/Google/Chrome/Default/` — Chrome profile
- `~/Library/Application Support/Firefox/Profiles/` — Firefox profile
- `~/.config/google-chrome/Default/` — Chrome profile on Linux
- `~/.mozilla/firefox/*.default/` — Firefox profile on Linux
- `$LOCALAPPDATA/Mozilla/Firefox/Profiles/` — Firefox profile on Windows (cache2/ subdirs are safe; the profile root is NOT)
- `$LOCALAPPDATA/Google/Chrome/User Data/Default/` — Chrome profile on Windows (NEVER delete; use Git Bash `$LOCALAPPDATA`)
- `$LOCALAPPDATA/Microsoft/Edge/User Data/Default/` — Edge profile on Windows
