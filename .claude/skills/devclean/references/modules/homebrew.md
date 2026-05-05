# Homebrew

platform: macos, linux
detect: which brew

## Scan

- `du -sh ~/Library/Caches/Homebrew` (macOS) or `~/.cache/Homebrew` (Linux)
- `brew cleanup --prune=all --dry-run 2>/dev/null | tail -5` — preview what would be removed
- `brew list --cask 2>/dev/null | wc -l` — number of installed casks
- `brew list 2>/dev/null | wc -l` — number of installed formulae

## Analysis

**Homebrew download cache:**
- Safe — old formula/cask downloads no longer needed after installation
- `brew cleanup` removes versions older than the configured prune days (default 120)
- `--prune=all` removes everything regardless of age

**Old formula versions:**
- Safe — Homebrew keeps the current and one previous version; `cleanup` removes older ones
- The dry-run output shows exact size to be freed before committing

## Cleanup

Safe | Remove all cached downloads and old formula versions | `brew cleanup --prune=all`

## NEVER Delete

- `/opt/homebrew/` (Apple Silicon) or `/usr/local/Homebrew/` (Intel) — the Homebrew installation itself
- `~/.Brewfile` or `~/Brewfile` — saved package list (user's package manifest)
