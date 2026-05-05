# Windows Package Managers (Chocolatey / Scoop)

platform: windows
detect: which choco 2>/dev/null || which scoop 2>/dev/null

## Scan

- `which choco 2>/dev/null && du -sh "${ChocolateyInstall:-C:/ProgramData/chocolatey}/cache"` — Chocolatey download cache (if exists)
- `which choco 2>/dev/null && du -sh "${ChocolateyInstall:-C:/ProgramData/chocolatey}/lib-bad"` — Chocolatey failed installs (if exists)
- `which scoop 2>/dev/null && du -sh ~/scoop/cache` — Scoop downloaded installer archive cache (if exists)
- `which scoop 2>/dev/null && find ~/scoop/apps -maxdepth 3 -name "old" -type d 2>/dev/null` — Scoop old version dirs retained after updates (if exists)
- `du -sh ~/scoop/apps` — total Scoop app install tree (if exists)

## Analysis

**Chocolatey download cache (%ChocolateyInstall%\cache or C:\ProgramData\chocolatey\cache):**
- Safe — downloaded .nupkg and installer archives; re-downloaded on next install/upgrade
- Typically small unless many large packages have been installed recently

**Chocolatey lib-bad (%ChocolateyInstall%\lib-bad):**
- Safe — leftover package files from failed or interrupted installs; no longer active

**Scoop cache (~/scoop/cache):**
- Safe — downloaded installer archives for each package version; re-downloaded on next install
- Grows with each `scoop update` if not cleared regularly

**Scoop old version directories (~/scoop/apps/*/old/):**
- Safe — previous versions kept by Scoop after an update for rollback purposes
- `scoop cleanup *` removes all old versions for all installed apps
- After cleanup, rollback to a previous version is no longer possible

**winget:**
- Minimal transparent cache; managed by Windows; not a significant space consumer
- No manual cleanup needed; noted here for completeness

## Cleanup

Safe   | Clear Chocolatey download cache | `choco cache remove` or `rm -rf "${ChocolateyInstall:-C:/ProgramData/chocolatey}/cache"/*`
Safe   | Remove Chocolatey failed-install leftovers | `rm -rf "${ChocolateyInstall:-C:/ProgramData/chocolatey}/lib-bad"/*`
Safe   | Clear Scoop download cache (all packages) | `scoop cache rm *`
Safe   | Remove Scoop old version directories (all apps) | `scoop cleanup *`

## NEVER Delete

- `${ChocolateyInstall}/bin/` — Chocolatey shims and the `choco` binary itself
- `${ChocolateyInstall}/lib/` — currently active installed package files (only `lib-bad\` is safe)
- `${ChocolateyInstall}/config/` — Chocolatey configuration
- `~/scoop/apps/<app>/current/` — the active version symlink target; only `old/` subdirectories are safe
- `~/scoop/shims/` — Scoop PATH shims; deleting breaks installed app commands
- `~/scoop/persist/` — user data persisted across Scoop app upgrades (config files, databases)
