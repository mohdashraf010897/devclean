# .NET / NuGet

platform: all
detect: which dotnet || test -d ~/.nuget

## Scan

- `du -sh "$USERPROFILE/.nuget/packages"` (Windows) or `du -sh ~/.nuget/packages` (macOS/Linux) — NuGet global package cache
- `du -sh "$LOCALAPPDATA/NuGet/v3-cache"` (Windows, if exists) — NuGet HTTP cache (modern .NET SDK 5+)
- `du -sh "$LOCALAPPDATA/NuGet/Cache"` (Windows, if exists) — NuGet HTTP cache (legacy NuGet client)
- `du -sh ~/.local/share/NuGet/Cache` (Linux, if exists) — NuGet HTTP cache
- `du -sh ~/Library/Caches/NuGet` (macOS, if exists) — NuGet HTTP cache
- `du -sh "C:/Program Files/dotnet/packs"` (Windows, if exists) — .NET SDK workload packs
- `find ~/projects -maxdepth 5 \( -name "bin" -o -name "obj" \) -type d 2>/dev/null | head -30` — per-project build output dirs (adjust root path as needed)
- `dotnet nuget locals all --list` — show all NuGet local cache paths and sizes (cross-platform)

## Analysis

**NuGet global package cache (~/.nuget/packages or %USERPROFILE%\.nuget\packages):**
- Safe — extracted package contents; re-downloaded and re-extracted on next restore
- Typically 5–20 GB for active .NET development; highest-value cleanup target
- Clearing forces a full package restore on next build (adds time but no data loss)

**NuGet HTTP cache:**
- Safe — downloaded .nupkg archives before extraction; fully redundant with global cache
- Separate path per OS (see Scan above)

**.NET SDK workload packs (C:\Program Files\dotnet\packs):**
- Review — contains workloads (e.g., MAUI, WASM, Android) for installed SDK versions
- Old SDK version subdirectories are safe to remove after upgrading
- Do not remove packs for the currently active SDK version
- Requires admin on Windows to delete

**Per-project bin/ and obj/ directories:**
- Review — safe to delete; MSBuild and dotnet CLI recreate them on next build
- Deleting triggers a full rebuild; only recommend for inactive or archived projects
- `dotnet clean` is the preferred approach (respects project config)

## Cleanup

Safe   | Clear all NuGet local caches (cross-platform, preferred) | `dotnet nuget locals all --clear`
Safe   | Clear NuGet global package cache manually (Windows) | `rm -rf "$USERPROFILE/.nuget/packages"/*`
Safe   | Clear NuGet global package cache manually (macOS/Linux) | `rm -rf ~/.nuget/packages/*`
Safe   | Clear NuGet HTTP cache manually (macOS) | `rm -rf ~/Library/Caches/NuGet`
Safe   | Clear NuGet HTTP cache manually (Linux) | `rm -rf ~/.local/share/NuGet/Cache`
Safe   | Clear NuGet HTTP cache manually (Windows) | `rm -rf "$LOCALAPPDATA/NuGet/v3-cache" "$LOCALAPPDATA/NuGet/Cache"`
Review | Clean build output for a project | `dotnet clean <project_or_sln_path>` or `rm -rf <project_path>/bin <project_path>/obj`

## NEVER Delete

- `~/.nuget/packages/<package>/<version>/` — do not delete individual package versions by hand; use `dotnet nuget locals` to clear atomically
- `C:/Program Files/dotnet/` (parent dir) — .NET SDK and runtime installation; only `packs\` subdirectory old-version entries are safe to remove
- `*.csproj`, `*.sln`, `*.fsproj` — project files
- `NuGet.Config`, `nuget.config` — feed configuration; deleting breaks package restore
- `.nuget/packages.config` — legacy package reference files
