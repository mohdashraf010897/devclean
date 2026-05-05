# Xcode

platform: macos
detect: test -d ~/Library/Developer/Xcode

## Scan

- `du -sh ~/Library/Developer/Xcode/DerivedData` (if exists)
- `du -sh ~/Library/Developer/Xcode/Archives` (if exists)
- `du -sh ~/Library/Developer/Xcode/iOS\ DeviceSupport` (if exists)
- `du -sh ~/Library/Developer/Xcode/watchOS\ DeviceSupport` (if exists)
- `ls -lt ~/Library/Developer/Xcode/Archives/` — list archive dates

## Analysis

**DerivedData:**
- Always Safe — build artifacts that Xcode regenerates on next build
- No data loss; only cost is rebuild time

**Archives:**
- Review — check dates; recent archives may be needed for App Store submission or crash symbolication
- Archives older than 6 months with no associated TestFlight/App Store submission → likely safe
- Always ask before deleting; user knows their release cadence

**iOS/watchOS DeviceSupport:**
- Review — symbols for physical devices; only needed when connecting that device version
- Old OS versions (3+ major versions behind current) are unlikely to be needed again
- Safe to delete for versions you no longer support or test on

**Risk classification:**
- DerivedData → Safe
- Archives → Review
- DeviceSupport for old OS versions → Review

## Cleanup

Safe   | Clear DerivedData | `rm -rf ~/Library/Developer/Xcode/DerivedData`
Review | Delete old archives (list first, confirm per archive) | `rm -rf ~/Library/Developer/Xcode/Archives/<date_folder>`
Review | Delete old device support | `rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/<version>`

## NEVER Delete

- `~/Library/Developer/Xcode/Archives/` for any version currently in TestFlight or the App Store — crash reports require the dSYM files inside
- `~/Library/Developer/Xcode/UserData/` — IDE preferences, key bindings, snippets
