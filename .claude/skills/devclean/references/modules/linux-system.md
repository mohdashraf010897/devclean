# Linux System Caches

platform: linux
detect: uname -s | grep -q Linux

## Scan

- `du -sh ~/.cache` — user-level XDG cache
- `du -sh /var/cache/apt` — APT package download cache (requires sudo awareness)
- `du -sh /var/cache/yum` or `/var/cache/dnf` — RPM package cache (if exists)
- `du -sh /var/cache/pacman/pkg` — Pacman package cache (Arch, if exists)
- `du -sh /var/log` — system logs
- `du -sh ~/.local/share/Trash` — user trash
- `find /tmp -maxdepth 1 -type f -mtime +7 2>/dev/null | wc -l` — stale temp files

## Analysis

**`~/.cache`:**
- Safe — XDG user cache; all apps store regeneratable data here
- Sub-directories may be app-specific; large entries worth noting individually

**APT/YUM/DNF/Pacman cache:**
- Safe — downloaded package archives; can be cleared after installation
- Requires root: note this clearly before suggesting cleanup
- APT: `apt-get clean` or `apt-get autoclean`
- DNF: `dnf clean all`
- Pacman: `pacman -Sc`

**System logs (`/var/log`):**
- Review — active logs should not be deleted; old rotated logs (`.gz`, `.1`, `.2`) are safe
- Use `journalctl --vacuum-time=30d` to clean systemd journal
- Requires root

**Trash:**
- Review — user explicitly moved these here; confirm before clearing

**`/tmp` stale files:**
- Safe — files older than 7 days are typically abandoned

## Cleanup

Safe   | Clear user XDG cache | `rm -rf ~/.cache/*`
Safe   | Clear APT cache (requires sudo) | `sudo apt-get clean`
Safe   | Clear DNF cache (requires sudo) | `sudo dnf clean all`
Safe   | Clear Pacman old packages (requires sudo) | `sudo pacman -Sc --noconfirm`
Safe   | Vacuum systemd journal older than 30d (requires sudo) | `sudo journalctl --vacuum-time=30d`
Review | Empty user trash | `rm -rf ~/.local/share/Trash/files/* ~/.local/share/Trash/info/*`
Safe   | Remove stale temp files (>7 days) | `find /tmp -maxdepth 1 -type f -mtime +7 -delete 2>/dev/null`

## NEVER Delete

- `/var/log/auth.log`, `/var/log/syslog` — active system logs (rotation handles them)
- `~/.local/share/` (wholesale) — application data, not just cache
- `/etc/` — system configuration
- `/var/lib/` — package manager state and application data
