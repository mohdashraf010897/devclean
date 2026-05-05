# Visual Studio

platform: windows
detect: test -d "$LOCALAPPDATA/Microsoft/VisualStudio"

## Scan

- `du -sh "$LOCALAPPDATA/Microsoft/VisualStudio"` — IDE caches for all installed VS versions (if exists)
- `du -sh "C:/ProgramData/Microsoft/VisualStudio/Packages"` — VS installer component cache (if exists)
- `find "$TEMP" -maxdepth 1 -name "VisualStudio*" -type d 2>/dev/null` — temp build artifacts in %TEMP%
- `du -sh "$LOCALAPPDATA/Temp"` — VS and MSBuild temp files (if exists)
- `find . -maxdepth 5 \( -name "bin" -o -name "obj" \) -type d 2>/dev/null | head -30` — per-project build output dirs (run from projects root)

## Analysis

**VS Installer component cache (C:\ProgramData\Microsoft\VisualStudio\Packages):**
- Safe — cached VSIX and component packages used by the VS Installer for repair/update; re-downloaded on demand
- Typically 5–20 GB; high-value cleanup target
- Clearing forces a slower repair/modify operation next time but causes no functional loss

**Per-version IDE caches ($LOCALAPPDATA\Microsoft\VisualStudio\<version>):**
- Safe — per-version caches such as component cache, MEF cache, IntelliSense DB
- Stale version directories (e.g., `16.0_*` dirs when only VS 2022 / `17.0_*` is installed) are also Safe
- Review old version dirs: if VS 2019 (16.x) is no longer installed, its cache dir is orphaned and safe to remove

**%TEMP%\VisualStudio* and %LOCALAPPDATA%\Temp:**
- Safe — leftover temp files from build and designer operations

**bin/ and obj/ directories (per-project build output):**
- Review — safe to delete (MSBuild rebuilds them), but triggers a full rebuild on next build
- Only recommend for projects not built recently (30+ days) or when disk space is critical
- See also: dotnet.md for cross-platform .NET build output guidance

## Cleanup

Safe   | Clear VS Installer component cache (requires admin or elevated prompt) | `rm -rf "C:/ProgramData/Microsoft/VisualStudio/Packages"/*`
Safe   | Clear current VS IDE caches (close VS first) | `rm -rf "$LOCALAPPDATA/Microsoft/VisualStudio"/*/ComponentModelCache "$LOCALAPPDATA/Microsoft/VisualStudio"/*/Cache`
Safe   | Remove orphaned VS 2019 IDE cache (if VS 2022 is the only installed version) | `rm -rf "$LOCALAPPDATA/Microsoft/VisualStudio"/16.0_*`
Safe   | Clear VS-related temp files | `rm -rf "$TEMP"/VisualStudio* "$LOCALAPPDATA/Temp"/VisualStudio*`
Review | Delete bin/ and obj/ build output in a project | `rm -rf <project_path>/bin <project_path>/obj`

## NEVER Delete

- `C:/Program Files/Microsoft Visual Studio/` — VS installation directory
- `C:/ProgramData/Microsoft/VisualStudio/` (parent dir) — VS installer metadata; only `Packages\` subdirectory is safe to clear
- `$LOCALAPPDATA/Microsoft/VisualStudio/*/Settings/` — per-version user settings and keybindings
- `$LOCALAPPDATA/Microsoft/VisualStudio/*/Extensions/` — installed extensions
- `.sln`, `.csproj`, `.vbproj`, `.vcxproj` — project and solution files
