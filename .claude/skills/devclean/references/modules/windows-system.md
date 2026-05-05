# Windows System Caches

platform: windows
detect: uname -s | grep -qi mingw || uname -s | grep -qi cygwin

## Scan

- `du -sh "$TEMP"` — user temp folder (%TEMP%)
- `du -sh "C:/Windows/Temp"` — system temp folder (requires admin)
- `du -sh "$USERPROFILE/AppData/Local/Microsoft/Windows/WER"` — Windows Error Reporting cache (if exists)
- `du -sh "C:/Windows/Prefetch"` — Prefetch files (requires admin, if exists)
- `du -sh "C:/Windows/SoftwareDistribution/Download"` — Windows Update download cache (requires admin, if exists)
- `find "$USERPROFILE" -name "\$Recycle.Bin" -maxdepth 3 -type d 2>/dev/null` — Recycle Bin per-drive (if exists)
- `du -sh "C:/\$Recycle.Bin"` (if exists)

## Analysis

**%TEMP% and C:\Windows\Temp:**
- Safe — temporary files left by installers, Windows components, and apps
- System Temp requires an elevated (admin) prompt to fully clear
- Files in use by running processes cannot be deleted; skip them silently

**Recycle Bin ($Recycle.Bin):**
- Review — user explicitly moved these files here; confirm before clearing
- Each drive has its own `$Recycle.Bin` directory

**Windows Update Download cache (C:\Windows\SoftwareDistribution\Download):**
- Safe — downloaded update packages already applied; Windows re-downloads if needed
- Requires admin: before suggesting any admin-required operation, note that the user must run Git Bash (or PowerShell/CMD) as Administrator; otherwise the command will silently fail or produce an Access Denied error
- Do not delete parent `SoftwareDistribution\` directory, only `Download\` subdirectory

**Windows Prefetch (C:\Windows\Prefetch):**
- Safe — launch-time hints regenerated automatically by Windows
- Requires admin
- Clearing causes slightly slower first launches after reboot; negligible on SSDs

**Windows Error Reporting (%LOCALAPPDATA%\Microsoft\Windows\WER):**
- Safe — crash dump and error report staging; already sent or waiting to be sent
- No user data; reports are anonymized telemetry

## Cleanup

Safe   | Clear user temp folder | `rm -rf "$TEMP"/*`
Safe   | Clear system temp folder (requires admin) | `rm -rf "C:/Windows/Temp"/*`
Safe   | Clear Windows Error Reporting cache | `rm -rf "$LOCALAPPDATA/Microsoft/Windows/WER/ReportQueue" "$LOCALAPPDATA/Microsoft/Windows/WER/ReportArchive"`
Safe   | Clear Windows Update download cache (requires admin) | `rm -rf "C:/Windows/SoftwareDistribution/Download"/*`
Safe   | Clear Prefetch (requires admin) | `rm -rf "C:/Windows/Prefetch"/*`
Review | Empty Recycle Bin (C: drive) | `rm -rf "C:/\$Recycle.Bin"/*`

## NEVER Delete

- `C:/Windows/` (wholesale) — OS files
- `C:/Windows/SoftwareDistribution/` (parent dir) — Windows Update state; only `Download\` subdirectory is safe
- `C:/Windows/System32/` — system binaries
- `$USERPROFILE/AppData/Local/` (wholesale) — application data and settings beyond WER
- `$USERPROFILE/AppData/Roaming/` — roaming application data including browser profiles and IDE settings
