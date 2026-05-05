# DevClean

[![GitHub stars](https://img.shields.io/github/stars/mohdashraf010897/devclean?style=flat-square)](https://github.com/mohdashraf010897/devclean/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![Made for Claude Code](https://img.shields.io/badge/Made%20for-Claude%20Code-blueviolet?style=flat-square)](https://docs.anthropic.com/en/docs/claude-code)

**AI-powered cross-platform developer workstation cleanup that actually understands your toolchain.**

## The Problem

Developer machines accumulate tens of gigabytes of hidden bloat -- emulator snapshots, orphaned system images, IDE backup databases, stale caches -- buried in `~/Library`, `~/.android`, and other places you never browse. Traditional cleanup tools treat these as opaque blobs. They can tell you a folder is large, but they can't tell you which Android system images are orphaned or which iOS simulators are duplicates. DevClean can.

## What DevClean Covers

- **Android** -- AVDs, snapshots, orphaned system images (cross-references `config.ini`)
- **iOS Simulators** -- per-version deduplication, booted vs. never-used detection
- **Xcode** -- DerivedData, Archives, iOS DeviceSupport
- **Node.js / Package Managers** -- Yarn, npm, Cypress, Playwright caches, `node_modules` hunting
- **Python** -- pip cache, pyenv download cache
- **Rust** -- Cargo registry cache and source
- **Java** -- Gradle caches, Maven local repository
- **Homebrew** -- download cache, old formula versions
- **IDEs** -- Cursor (`state.vscdb.backup`), VS Code, JetBrains caches
- **Visual Studio** -- component and NuGet caches (Windows)
- **.NET / NuGet** -- NuGet HTTP and global packages cache
- **Docker** -- images, containers, build cache, volumes
- **Browser Caches** -- Chrome, Firefox, Edge (macOS, Linux, and Windows paths)
- **Linux System** -- apt/dnf/pacman package caches, systemd journal, thumbnail cache
- **Windows System** -- Temp folders, Windows Update cache, Prefetch, Event Logs
- **WSL** -- WSL distro caches and package manager caches from within Windows
- **Windows Package Managers** -- Chocolatey, Scoop, winget caches

## How It Works

DevClean follows a strict four-phase workflow:

1. **Detect + Load** -- detects OS and shell environment, then loads only the modules relevant to the current platform and installed toolchains
2. **Scan + Analyze** -- walks known developer tool paths, reads config files, cross-references components, and risk-rates every finding
3. **Confirm** -- presents a numbered cleanup plan sorted by size and waits for explicit approval
4. **Execute + Verify** -- runs approved deletions and reports actual space recovered per item

Nothing is ever deleted without your explicit confirmation. There is no `--force` flag.

## Installation

**Prerequisites:** [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and working on macOS, Linux, or Windows.

```bash
# Clone the repo
git clone https://github.com/mohdashraf010897/devclean.git

# For global access (available in any project):
cp -r devclean/.claude/skills/devclean ~/.claude/skills/

# Or for a single project:
cp -r devclean/.claude/skills/devclean your-project/.claude/skills/
```

That's it. No build step, no dependencies.

**Usage:**

```bash
# Launch Claude Code
claude

# Run DevClean
/devclean
```

## Example Output

```
Scanning your development environment...

Found 9 cleanup opportunities across 6 categories:

 ANDROID EMULATORS                                    34.2 GB
 ├── AVD: Pixel_6_API_33          8.1 GB  (last booted: 2024-11-02)
 ├── AVD: Pixel_7_API_33          8.3 GB  (last booted: 2025-01-15)
 ├── AVD: Pixel_4_API_30          7.4 GB  (last booted: 2024-06-18)
 └── Orphaned system images      10.4 GB  (no AVD references these)

 iOS SIMULATORS                                        8.1 GB
 ├── iOS 15.5: 6 devices (recommend keep 1)
 ├── iOS 16.0: 8 devices (recommend keep 1)
 ├── iOS 16.4: 12 devices (recommend keep 1)
 └── iOS 18.0: 10 devices (recommend keep 1)

 IDE & EDITOR CACHES                                  23.8 GB
 ├── Cursor state.vscdb.backup   21.0 GB   [SAFE]
 └── JetBrains caches             2.8 GB   [SAFE]

 PACKAGE MANAGER CACHES                                8.7 GB
 ├── Yarn global cache            5.6 GB   [SAFE]
 └── Cypress binary cache         3.1 GB   [REBUILD: ~3 min]

Cleanup Plan:
 1. [Safe]   Delete Cursor state.vscdb.backup        ~21.0 GB
 2. [Safe]   Delete Yarn cache                        ~5.6 GB
 3. [Safe]   Delete Cypress cache                     ~3.1 GB
 4. [Safe]   Delete JetBrains caches                  ~2.8 GB
 5. [Review] Remove orphaned Android system images   ~10.4 GB
 6. [Review] Delete 32 duplicate iOS simulators       ~6.2 GB

 Total potential recovery: ~49.1 GB

Which items to clean? (e.g., "all", "1,2,3,4", "all safe")
```

## Safety

DevClean is built around a "scan first, delete never (until you say so)" philosophy:

- **Two-phase confirmation** -- you see the full plan before anything happens. For "Review" items, DevClean asks for confirmation a second time.
- **Risk ratings** -- every item is classified as Safe (caches that rebuild automatically), Review (needs analysis, e.g. duplicate AVDs), or Risky (contains user data, flagged but never offered for deletion).
- **NEVER-delete list** -- browser profiles, SSH keys, keychains, live IDE state databases, `.env` files, and user documents are hardcoded as off-limits. DevClean will not offer to remove them under any circumstances.

## Requirements

- macOS, Linux, or Windows (Windows requires [Git Bash](https://git-scm.com/downloads) for Claude Code)
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)

## Contributing

Contributions are welcome. If you want to add support for a new toolchain, fix a bug, or improve the safety reference:

1. Fork the repo
2. Create a feature branch
3. Submit a pull request

If you find a cleanup category that's missing or a safety edge case, please open an issue.

## License

[MIT](LICENSE)

## Credits

Built by [@mohdashraf010897](https://github.com/mohdashraf010897). Powered by [Claude Code](https://docs.anthropic.com/en/docs/claude-code).
