# WSL2 (Windows Subsystem for Linux)

platform: windows
detect: which wsl.exe 2>/dev/null && wsl --list >/dev/null 2>&1

## Scan

- `wsl --list --verbose` (if wsl.exe is in PATH) — list installed distros and their state
- `find "$LOCALAPPDATA/Packages" -name "ext4.vhdx" 2>/dev/null` — locate all WSL2 virtual disk images
- `du -sh "$LOCALAPPDATA/Packages/CanonicalGroupLimited.Ubuntu"*/LocalState/ext4.vhdx 2>/dev/null` — Ubuntu .vhdx size (if exists)
- `du -sh "$LOCALAPPDATA/Packages/TheDebianProject.DebianGNULinux"*/LocalState/ext4.vhdx 2>/dev/null` — Debian .vhdx size (if exists)
- `wsl -d <distro> -- df -h /` — actual disk usage inside a running distro vs allocated .vhdx size
- `wsl -d <distro> -- du -sh ~/.cache ~/.npm ~/.nuget 2>/dev/null` — cache dirs inside the distro (if exists)

## Analysis

**WSL2 .vhdx disk images:**
- Review — the virtual disk grows as files are added inside WSL but never auto-shrinks when files are deleted
- The gap between `.vhdx` file size (host) and `df -h /` used space (inside distro) is compactable space
- Example: a 40 GB `.vhdx` where `df` shows 8 GB used means 32 GB can be reclaimed by compaction
- Compaction requires: (1) WSL shutdown (`wsl --shutdown`), (2) Hyper-V or admin access, (3) `diskpart` or PowerShell `Optimize-VHD`
- Note: compaction does not delete data; it only reclaims already-freed space inside the virtual disk

**Cache and build dirs inside WSL:**
- Review — `~/.cache`, `node_modules`, `~/.nuget/packages`, Python venvs, and Rust target dirs inside WSL all consume .vhdx space
- Best cleaned from inside WSL using the relevant DevClean modules (node.md, python.md, rust.md, dotnet.md)
- Cleaning inside WSL first, then compacting the .vhdx, gives the full benefit

**Multiple distros:**
- Each distro has its own independent .vhdx; scan all of them
- Unused distros (e.g., an old Ubuntu 20.04 kept after upgrading to 22.04) can be unregistered entirely with `wsl --unregister <distro>` — this permanently deletes that distro's .vhdx

## Cleanup

Review | Shut down all WSL distros before compaction | `wsl --shutdown`
Review | Compact Ubuntu .vhdx via PowerShell (requires admin/Hyper-V) | `Optimize-VHD -Path "$env:LOCALAPPDATA\Packages\CanonicalGroupLimited.Ubuntu_...\LocalState\ext4.vhdx" -Mode Full`
Review | Compact .vhdx via diskpart (requires admin) | `diskpart` → `select vdisk file="<path to ext4.vhdx>"` → `compact vdisk`
Risky  | Unregister and permanently delete an unused distro | `wsl --unregister <DistroName>`

## NEVER Delete

- `ext4.vhdx` directly via file explorer or `rm` — this permanently destroys the entire Linux installation and all data inside it; always use `wsl --unregister` if removal is intended
- `$LOCALAPPDATA/Packages/CanonicalGroupLimited.Ubuntu*/` (parent dir) — WSL distro app package; only compact the .vhdx inside it
- Any `.vhdx` file while WSL is running — always run `wsl --shutdown` first
