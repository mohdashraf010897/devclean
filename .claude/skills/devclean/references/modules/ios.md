# iOS Simulators

platform: macos
detect: which xcrun && xcrun simctl list devices >/dev/null 2>&1

## Scan

- `xcrun simctl list devices available` — all simulators with UUIDs
- `du -sh ~/Library/Developer/CoreSimulator/Devices` — total device storage
- `du -sh ~/Library/Developer/CoreSimulator/Caches` — dyld shared caches per runtime
- `du -sh ~/Library/Developer/CoreSimulator/Devices/<UUID>` for the 10 largest devices

## Analysis

**Deduplication:**
- Group simulators by iOS major version (e.g., all iOS 16.x together)
- Within each version, count devices; note which have been booted (folder > 20 MB) vs never-used (≤ 20 MB)
- More than one device per iOS version = duplicates → recommend keeping one representative iPhone per version (prefer newest available model)
- Never-booted devices at 17 MB each contribute little individually; flag in bulk

**Runtime caches (`CoreSimulator/Caches/dyld/`):**
- Each subfolder corresponds to a build ID tied to a specific iOS runtime
- If the user keeps at least one simulator for a runtime, its dyld cache is needed
- If ALL simulators for a runtime are deleted, its dyld cache is orphaned → Safe to delete AFTER simulators are gone

**Risk classification:**
- Individual simulator deletion → Review (can be recreated from Xcode, but booted simulators lose installed apps/data)
- CoreSimulator dyld cache for runtime with no remaining simulators → Safe
- CoreSimulator dyld cache for active runtime → Risky (slow 5–10 min rebuild on next boot)

## Cleanup

Review | Delete a specific simulator | `xcrun simctl delete <UUID>`
Review | Delete all unavailable simulators | `xcrun simctl delete unavailable`
Safe   | Delete dyld cache for removed runtime | `rm -rf ~/Library/Developer/CoreSimulator/Caches/dyld/<build_id>`

## NEVER Delete

- `~/Library/Developer/CoreSimulator/Caches/dyld/<id>` while any simulator using that runtime still exists
- Simulators marked as Booted (running) — must shut down first
