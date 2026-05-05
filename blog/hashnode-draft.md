---
title: "I Freed 45 GB From My MacBook With One Claude Command"
subtitle: "How I built DevClean — an AI-powered Claude Code skill that actually understands your dev toolchain"
slug: devclean-ai-powered-macos-developer-cleanup
tags: macos, linux, windows, developer-tools, claude, ai, productivity
canonical_url: ""
cover_image: <!-- TODO: Add cover image URL — suggested: terminal screenshot showing DevClean scan output -->
publishAs: ""
series: ""
---

# I Freed 45 GB From My MacBook With One Claude Command

Last week, my MacBook told me I had 14 GB of free space. I'm running a 512 GB drive. Where did it all go?

I opened Finder, sorted by size, and found the usual suspects — but what shocked me was *where* the bloat actually lived. Not in my Downloads folder. Not in forgotten video files. It was hiding inside developer tools I use every single day.

Cursor's `state.vscdb.backup` file alone was **21 GB**. One file. A SQLite backup from my code editor.

That kicked off a three-hour deep dive that changed how I think about developer workstation maintenance — and ended with me building a tool to make sure I never have to do it manually again.

## The Hidden Cost of Being a Developer

Here's what I found when I actually looked:

| Category | Size | What it was |
|---|---|---|
| Cursor state backup | 21 GB | `state.vscdb.backup` — a single SQLite file |
| Android emulators | 34 GB | 3 AVDs + orphaned system images nobody was using |
| iOS simulators | ~8 GB | 36 devices across iOS 15.5, 16.0, 16.4, 18.0 — mostly duplicates |
| Yarn cache | 5.6 GB | Global package cache from hundreds of `yarn install` runs |
| Cypress cache | 3.1 GB | Old Electron binaries for end-to-end testing |
| JetBrains caches | 2.8 GB | IntelliJ/WebStorm index and log files |

**Total recovered: ~45 GB.**

And this isn't unusual. If you're a mobile developer or full-stack engineer on macOS, you're probably sitting on 50-100 GB of dev tool bloat right now. You just don't know it because these files hide in `~/Library`, `~/.android`, and `~/Library/Developer/CoreSimulator` — places you never browse.

## Why Existing Tools Don't Cut It

I tried the obvious solutions first.

**CleanMyMac X** ($40/year) found some caches, but it treats developer tools as opaque blobs. It doesn't know that my three Android AVDs all target API 33 and share the same system image — or that two of them haven't been booted in six months. It just sees "big folder, want to delete?"

**DaisyDisk** is great at showing you *what's big*. But it has zero opinion on *what's safe to remove*. It'll happily show you that `~/.android/avd` is 34 GB, but the decision-making is entirely on you. Hope you remember which system images are still referenced by your AVD configs.

**DevCleaner for Xcode** covers exactly one tool — Xcode. It clears derived data and old device support files. Useful, but it doesn't touch your Android setup, your Node caches, or your IDE bloat.

Here's the fundamental problem: **these tools don't understand developer toolchains at a semantic level.**

They can't read an AVD's `config.ini` to figure out which system images are actively referenced and which are orphaned. They can't look at your iOS simulators and realize you have eight iPhone 14 Pro devices across four iOS versions when you only need one per version. They can't assess risk — is deleting this cache going to cost you a 20-minute rebuild, or is it completely safe?

I needed something smarter.

## Introducing DevClean

DevClean is a Claude Code skill that performs intelligent, developer-aware cleanup of your developer workstation (macOS, Linux, and Windows). It's not a compiled binary or an Electron app. It's an AI skill that runs inside Claude Code, which means it can actually *reason* about what it finds.

The workflow is straightforward:

```
You: /devclean

DevClean: Scanning your development environment...

Found 7 cleanup opportunities across 5 categories:

 ANDROID EMULATORS                                    34.2 GB
 ├── AVD: Pixel_6_API_33          8.1 GB  (last booted: 2024-11-02)
 ├── AVD: Pixel_7_API_33          8.3 GB  (last booted: 2025-01-15)
 ├── AVD: Pixel_4_API_30          7.4 GB  (last booted: 2024-06-18)
 └── Orphaned system images       10.4 GB  (no AVD references these)

 iOS SIMULATORS                                        8.1 GB
 ├── iOS 15.5: 6 devices (recommend keep 1)
 ├── iOS 16.0: 8 devices (recommend keep 1)
 ├── iOS 16.4: 12 devices (recommend keep 1)
 └── iOS 18.0: 10 devices (recommend keep 1)

 IDE & EDITOR CACHES                                  23.8 GB
 ├── Cursor state.vscdb.backup    21.0 GB   [SAFE]
 └── JetBrains caches              2.8 GB   [SAFE]

 PACKAGE MANAGER CACHES                                8.7 GB
 ├── Yarn global cache             5.6 GB   [SAFE]
 └── Cypress binary cache          3.1 GB   [REBUILD: ~3 min]

Risk Legend:  [SAFE] = no rebuild cost
              [REBUILD: Xm] = will need re-download on next use
```

But the interesting part isn't the scan. It's what happens next.

## Intelligence, Not Just Inventory

When I told DevClean to clean up my iOS simulators, I said: "keep one representative device per iOS version." It didn't just blindly delete random simulators. It analyzed the device types, picked the most commonly used form factor for each iOS version, and presented its reasoning:

```
You: Clean up the iOS simulators. Keep one per iOS version.

DevClean: Here's my plan:

 iOS 15.5 — keeping iPhone 13 Pro (most common test target)
           removing: iPhone 13, iPhone 13 mini, iPhone SE 3rd gen,
                     iPad Air 5th gen, iPad mini 6th gen

 iOS 16.0 — keeping iPhone 14 Pro
           removing: iPhone 14, iPhone 14 Plus, iPhone 14 Pro Max,
                     iPhone SE 3rd gen, iPad Air 5th gen,
                     iPad Pro 11-inch, iPad Pro 12.9-inch

 [... similar for iOS 16.4 and 18.0]

 Total simulators to remove: 32
 Estimated space freed: ~6.2 GB

 Proceed? [y/N]
```

This is the key differentiator. DevClean doesn't just find big files. It makes **judgment calls** — the kind a senior developer would make — and explains its reasoning before touching anything.

For Android, it reads each AVD's `config.ini` and `hardware-qemu.ini` to trace exactly which system images are in use. Only images with zero references get flagged as orphaned. It knows the difference between "this AVD uses API 33 x86_64" and "this system image for API 33 arm64-v8a is sitting here unused."

For package manager caches, it checks when each cache was last accessed and estimates the rebuild cost. Deleting your Yarn cache is free — `yarn install` will re-fetch in seconds. Deleting your Cypress cache means a ~3-minute Electron binary download next time you run tests. You should know that before you say yes.

## How It Works Under the Hood

DevClean runs as a Claude Code custom skill. That means it has access to Claude's reasoning capabilities while operating within your terminal. The architecture is simple:

1. **Scan** — Walks known developer tool paths (`~/.android/avd`, `~/Library/Developer/CoreSimulator`, `~/Library/Caches`, etc.) and catalogs what it finds
2. **Analyze** — Reads configuration files, checks cross-references between components, calculates last-used timestamps, and risk-rates every item
3. **Recommend** — Presents a prioritized cleanup plan with estimated space savings and risk levels
4. **Confirm** — Nothing is deleted without explicit approval. Every action is explained before execution
5. **Clean** — Executes the approved deletions and reports the actual space recovered

The "analyze" step is where the AI reasoning matters. A shell script can tell you a folder is 8 GB. It takes actual understanding of Android's emulator architecture to know that deleting `system-images/android-30/google_apis/x86_64` is safe because no AVD in `~/.android/avd/` references API 30 x86_64 anymore.

## Getting Started

DevClean is a Claude Code skill, so setup takes about 30 seconds.

**Prerequisites:** You need [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working.

**Installation:**

```bash
# Clone the repo
git clone https://github.com/mohdashraf010897/devclean.git

# Copy the skill to your Claude Code skills directory
cp -r devclean/.claude/skills/devclean ~/.claude/skills/

# That's it. No build step, no dependencies.
```

**Usage:**

```bash
# Launch Claude Code in any directory
claude

# Run DevClean
/devclean
```

DevClean runs a full scan, presents everything it finds, and lets you pick what to clean. You can tell it things like "only clean the safe items" or "skip the Android stuff" — it's a conversation, not a CLI with flags. There's no `--force` mode. That's intentional.

## What DevClean Doesn't Do

Being honest about limitations:

- **It's cross-platform.** macOS, Linux, and Windows (via Git Bash) are all supported. Platform-specific modules load automatically — you only see what's relevant to your machine.
- **It doesn't run on a schedule.** You invoke it when you want it. There's no background daemon.
- **Docker cleanup is conservative.** It scans Docker usage and offers `docker system prune`, but won't touch volumes without explicit confirmation due to data loss risk.
- **It requires Claude Code.** If you're not already using Claude Code as your AI coding assistant, this skill won't work standalone.

## What's Next

The roadmap for DevClean includes:

- **Cross-platform shipped** — macOS, Linux, and Windows modules are live. 19 toolchain modules total.
- **CI cache analysis** — Connect to your GitHub Actions or CircleCI caches and identify what's actually being used versus just accumulating.
- **Homebrew audit** — Find formulae and casks you installed once for a side project and never touched again.
- **Workspace-aware cleanup** — Tie cleanup recommendations to your active projects. If you haven't opened a React Native project in 6 months, maybe those specific emulator configs can go.
- **Scheduled scans** — Monthly cleanup reminders with delta reports showing what grew since last scan.

## Try It

DevClean is open source and free.

**GitHub:** [github.com/mohdashraf010897/devclean](https://github.com/mohdashraf010897/devclean)

If you're a developer on macOS, run a quick scan. You'll probably be surprised at what you find. I was sitting on 45 GB of recoverable space and had no idea — and I *actively* try to keep my machine clean.

If you find it useful, star the repo and share it with your team. If you find a bug or want to add a new cleanup category, PRs are welcome.

The best part about building this as a Claude Code skill rather than a traditional app: adding support for a new tool category doesn't require writing a parser. You describe the tool's file layout and configuration format, and the AI figures out the rest. That's a genuinely different approach to developer tooling — and I think we're only scratching the surface of what's possible here.

---

*Built by [Mohd Ashraf](https://github.com/mohdashraf010897). If you have questions or want to contribute, open an issue on the repo or find me on Twitter/X.*
