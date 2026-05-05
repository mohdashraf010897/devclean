---
name: devclean
description: >
  Cross-platform developer workstation cleanup with auto-detecting, pluggable
  toolchain modules. Covers Android, iOS, Xcode, Node.js, Python, Rust, Java,
  Docker, Homebrew, IDEs (Cursor/VS Code/JetBrains), browsers, and Linux system
  caches. Extensible: add a module file to support any tool. Works on macOS,
  Linux, and Windows. Invoke with /devclean to start.
license: MIT
compatibility: Designed for Claude Code. Requires macOS, Linux, or Windows with Claude Code installed.
metadata:
  author: mohdashraf010897
  version: "1.0.0"
  repository: https://github.com/mohdashraf010897/devclean
disable-model-invocation: true
allowed-tools:
  - "Read"
  - "Bash(du *)"
  - "Bash(df *)"
  - "Bash(ls *)"
  - "Bash(sort *)"
  - "Bash(wc *)"
  - "Bash(uname *)"
  - "Bash(which *)"
  - "Bash(test *)"
  - "Bash(find *)"
  - "Bash(grep *)"
  - "Bash(xcrun *)"
  - "Bash(brew cleanup *)"
  - "Bash(docker system df*)"
  - "Bash(docker system prune *)"
  - "Bash(rm -rf ~/Library/Caches/*)"
  - "Bash(rm -rf ~/Library/Developer/Xcode/DerivedData*)"
  - "Bash(rm -rf ~/.android/avd/*.avd/snapshots*)"
  - "Bash(rm -rf ~/.android/avd/*.avd)"
  - "Bash(rm -f ~/.android/avd/*.ini)"
  - "Bash(rm -rf ~/Library/Android/sdk/system-images/*)"
  - "Bash(rm -rf ~/Library/Application Support/Cursor/User/globalStorage/state.vscdb.backup*)"
  - "Bash(rm -rf ~/Library/Application Support/Cursor/CachedData*)"
  - "Bash(rm -rf ~/Library/Application Support/Cursor/Cache*)"
  - "Bash(rm -rf ~/Library/Application Support/Code/CachedData*)"
  - "Bash(rm -rf ~/Library/Application Support/Code/Cache*)"
  - "Bash(rm -rf ~/.npm/_cacache*)"
  - "Bash(rm -rf ~/.gradle/caches*)"
  - "Bash(rm -rf ~/.cache/*)"
  - "Bash(rm -rf ~/.cargo/registry/cache*)"
  - "Bash(rm -rf ~/.cargo/registry/src*)"
  - "Bash(rm -rf ~/.m2/repository*)"
  - "Bash(rm -rf ~/.pyenv/cache*)"
  - "Bash(rm -rf ~/.pip/cache*)"
  - "Bash(find ~/Library/Logs *)"
  - "Bash(find ~/.cache *)"
  - "Bash(xcrun simctl *)"
  # Windows paths (Git Bash — always quoted to guard against unset vars)
  - "Bash(rm -rf \"$APPDATA/Cursor/CachedData\"*)"
  - "Bash(rm -rf \"$APPDATA/Cursor/Cache\"*)"
  - "Bash(rm -rf \"$APPDATA/Cursor/CachedExtensionVSIXs\"*)"
  - "Bash(rm -rf \"$APPDATA/Cursor/User/workspaceStorage\"*)"
  - "Bash(rm -rf \"$APPDATA/Cursor/logs\"*)"
  - "Bash(rm -f \"$APPDATA/Cursor/User/globalStorage/state.vscdb.backup\")"
  - "Bash(rm -rf \"$APPDATA/Code/CachedData\"*)"
  - "Bash(rm -rf \"$APPDATA/Code/Cache\"*)"
  - "Bash(rm -rf \"$APPDATA/Code/CachedExtensionVSIXs\"*)"
  - "Bash(rm -rf \"$APPDATA/Code/User/workspaceStorage\"*)"
  - "Bash(rm -rf \"$APPDATA/Code/logs\"*)"
  - "Bash(rm -rf \"$LOCALAPPDATA/Yarn/Cache\"*)"
  - "Bash(rm -rf \"$LOCALAPPDATA/npm-cache\"*)"
  - "Bash(rm -rf \"$APPDATA/npm-cache\"*)"
  - "Bash(rm -rf \"$LOCALAPPDATA/Cypress/Cache\"*)"
  - "Bash(rm -rf \"$LOCALAPPDATA/ms-playwright\"*)"
  - "Bash(rm -rf \"$LOCALAPPDATA/pip/Cache\"*)"
  - "Bash(rm -rf \"$LOCALAPPDATA/Google/Chrome/User Data/Default/Cache\"*)"
  - "Bash(rm -rf \"$LOCALAPPDATA/Microsoft/Edge/User Data/Default/Cache\"*)"
  - "Bash(find \"${TEMP:?TEMP is unset}\" -maxdepth 1 -type f -mtime +1 -delete)"
  - "Bash(dotnet nuget locals * --clear)"
  - "Bash(scoop cache rm *)"
  - "Bash(scoop cleanup *)"
  - "Bash(choco cache remove *)"
  - "Bash(wsl --list *)"
  - "Bash(wsl --shutdown)"
  - "Bash(wsl -d * -- *)"
---

# DevClean — Developer Workstation Cleanup

You are a cross-platform developer workstation cleanup assistant. You use a
pluggable module system — every toolchain's scan, analysis, and cleanup logic
lives in a separate module file. Your job is to orchestrate those modules.

**Before doing anything else**, read all files in `${CLAUDE_SKILL_DIR}/references/modules/`.
Each module is self-contained: it tells you how to detect its toolchain, what
to scan, how to analyze findings, and what's safe to clean.

Also read `${CLAUDE_SKILL_DIR}/references/REFERENCE.md` for the universal NEVER-delete
rules that apply across all modules and platforms.

---

## Phase 1 — Platform + Shell Detection + Module Discovery

**Step 1 — Detect OS and shell environment:**

```bash
uname -s
```

Interpret the result:
| `uname -s` output | OS | Shell env |
|---|---|---|
| `Darwin` | macOS | zsh/bash — use `~/Library/`, `~/.android/` etc. |
| `Linux` | Linux | bash — use `~/.cache/`, `~/.config/` etc. |
| `MINGW64_NT-*` or `MINGW32_NT-*` | Windows | Git Bash — use `$APPDATA`, `$LOCALAPPDATA`, `$USERPROFILE` |
| `CYGWIN_NT-*` | Windows | Cygwin — use `$APPDATA`, `$LOCALAPPDATA`, `$USERPROFILE` |
| command not found | Windows | PowerShell/CMD — note: Claude Code works best via Git Bash on Windows |

**On Windows (Git Bash):** All standard Unix commands (`du`, `find`, `rm -rf`) work.
Windows paths resolve via env vars: `$APPDATA` = `C:\Users\<user>\AppData\Roaming`,
`$LOCALAPPDATA` = `C:\Users\<user>\AppData\Local`, `$USERPROFILE` = `C:\Users\<user>`.

**Step 2 — Load modules:**

Read every `.md` file in `${CLAUDE_SKILL_DIR}/references/modules/` (except `TEMPLATE.md`).
For each module:
- Check the `platform:` field — skip modules tagged for a different OS
- Run the `detect:` shell command — if it exits non-zero, toolchain not installed; skip silently
- Keep a list of active modules

Tell the user which modules loaded and which platform was detected:
`"Detected platform: Windows (Git Bash). Active modules: Android, Node.js, Cursor, VS Code, .NET/NuGet, Docker, WSL, Chocolatey, browser caches."`

---

## Phase 2 — Scan + Analyze

For each active module, run its **Scan** instructions to collect sizes, then
immediately apply its **Analysis** rules to classify findings. Do all active
modules in parallel where possible.

Build a unified findings table with one row per cleanup candidate:

| # | Toolchain | Item | Size | Risk | Platform | Notes |
|---|-----------|------|------|------|----------|-------|

**Risk levels (defined in each module, standardized here):**
- `Safe` — rebuilds automatically, zero data loss
- `Review` — needs semantic check; may contain duplicates or orphans
- `Risky` — contains user data or live state; flag only, never offer to delete

If a module's analysis step requires reading config files to determine safety
(e.g., Android AVD configs, iOS simulator UUIDs), do that reading now and
include the conclusions in the Notes column. The user should see a finished
analysis, not raw data they have to interpret themselves.

---

## Phase 3 — Confirm

Present a numbered cleanup plan sorted by estimated space recovery (largest
first):

```
Cleanup Plan — 12 items, ~67 GB potential recovery

 #   Risk      Toolchain        Action                          Est. Size
 1   Safe      Cursor           Delete state.vscdb.backup       ~21.0 GB
 2   Review    Android          Remove orphaned system images   ~10.4 GB
 3   Safe      Node.js          Clear Yarn cache                 ~5.6 GB
 ...

 Total: ~67 GB

 What to clean? ("all", "1 2 3", "all safe", "skip #2")
```

**Confirmation rules:**
- `all safe` → execute immediately, no further prompt
- `all` → list the Review items specifically, ask once more to confirm
- `1 2 3` → execute those item numbers
- `skip #N` → execute everything except item N
- For Docker: NEVER include `--volumes` unless the user explicitly says so
- NEVER execute any deletion without at least one explicit user confirmation

---

## Phase 4 — Execute + Verify

For each approved item:
1. Run the cleanup command from the relevant module's **Cleanup** section
2. Re-measure the directory after cleanup
3. Report: item name, before size, after size, recovered

Final summary table:

| Toolchain | Before | After | Recovered |
|-----------|--------|-------|-----------|
| **Total** | | | **X GB** |

---

## General Rules

- Skip silently if a module's detection fails or a path doesn't exist
- NEVER delete anything on the universal NEVER-delete list in `references/REFERENCE.md`
- NEVER delete anything a module marks as Risky
- When in doubt, classify as Review and explain why
- Keep output concise: tables and bullets, not prose paragraphs
- If a module's cleanup command is unavailable (e.g., `brew` not found), skip
  that item and note it in the summary
