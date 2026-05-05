# Android

platform: macos, linux, windows
detect: test -d ~/.android/avd || test -d "$USERPROFILE/.android/avd"

## Scan

- `du -sh ~/.android/avd` — total AVD size (macOS/Linux; on Windows use `$USERPROFILE/.android/avd`)
- For each `~/.android/avd/*.avd/` (macOS/Linux) or `$USERPROFILE/.android/avd/*.avd/` (Windows):
  - `du -sh <avd_dir>` — total size
  - `du -sh <avd_dir>/snapshots` — snapshot size (if exists)
  - Read `<avd_dir>/config.ini` → extract `image.sysdir.1`, `hw.device.name`, `avd.ini.displayname`
  - Read `~/.android/avd/<name>.ini` → extract `target` (API level)
  - Note: config.ini analysis logic is identical on Windows; only the path prefix differs (`$USERPROFILE/.android/` instead of `~/.android/`)
- Resolve SDK root: check `$ANDROID_HOME` or `$ANDROID_SDK_ROOT` env vars first; fall back to platform defaults:
  - macOS default: `~/Library/Android/sdk/`
  - Linux default: `~/Android/Sdk/`
  - Windows default: `$LOCALAPPDATA/Android/Sdk/`
- `du -sh <sdk_root>/system-images` — system images per variant

## Analysis

**AVD duplicates:**
- Group AVDs by `target` (API level) + `tag.id` (image variant)
- Same API + same image type + different `hw.device.name` = duplicate; recommend keeping one
- AVDs whose `snapshots/` is > 500 MB are good cleanup candidates regardless of duplicates

**Orphaned system images:**
- Collect all `image.sysdir.1` values from every AVD's config.ini
- List all directories under `sdk/system-images/<api>/<variant>/`
- Any variant directory not referenced by any AVD = orphaned → Review, safe to delete
- Different sub-variants are NOT interchangeable: `google_apis_playstore` ≠ `google_apis` ≠ `google_apis_ps16k`

**Risk classification:**
- AVD snapshots → Review (emulator cold-boots next time; userdata in `userdata-qemu.img.qcow2` is unaffected)
- Entire AVD → Review (irreversible; confirm the user no longer needs this device profile)
- Orphaned system images → Review (verify cross-reference before deleting)

## Cleanup

Review | Delete AVD snapshots (keeps emulator + userdata) (macOS/Linux) | `rm -rf ~/.android/avd/<NAME>.avd/snapshots`
Review | Delete AVD snapshots (Windows, Git Bash) | `rm -rf "$USERPROFILE/.android/avd/<NAME>.avd/snapshots"`
Review | Delete entire AVD (macOS/Linux) | `rm -rf ~/.android/avd/<NAME>.avd && rm -f ~/.android/avd/<NAME>.ini`
Review | Delete entire AVD (Windows, Git Bash) | `rm -rf "$USERPROFILE/.android/avd/<NAME>.avd" && rm -f "$USERPROFILE/.android/avd/<NAME>.ini"`
Review | Delete orphaned system image (all platforms) | `rm -rf <sdk_root>/system-images/<api>/<variant>` (substitute resolved SDK root from Scan step)

## NEVER Delete

- `~/.android/adbkey`, `~/.android/adbkey.pub` — ADB device authentication
- `~/.android/avd/<NAME>.avd/userdata-qemu.img.qcow2` — installed apps + data inside the emulator
- `<sdk_path>/system-images/<api>/<variant>/` when ANY AVD references it
