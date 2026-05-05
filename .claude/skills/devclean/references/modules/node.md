# Node.js / Package Managers

platform: all
detect: which node || which npm || which yarn || which pnpm

## Scan

- `du -sh ~/Library/Caches/Yarn` (macOS) or `~/.yarn/cache` (Linux) — Yarn v1 global cache
- `du -sh ~/.yarn/cache` — Yarn Berry (v2+) global cache
- `du -sh ~/.npm/_cacache` — npm cache (macOS/Linux)
- `du -sh "$APPDATA/npm-cache"` — npm cache (Windows, Git Bash; standard default location)
- `du -sh "$LOCALAPPDATA/npm-cache"` — npm cache (Windows, Git Bash; non-standard, only present if user ran `npm config set cache` to this path)
- `du -sh ~/.pnpm-store` or `~/.local/share/pnpm/store` — pnpm content-addressable store (macOS/Linux)
- `du -sh "$LOCALAPPDATA/pnpm/store"` — pnpm store (Windows, Git Bash)
- `du -sh ~/Library/Caches/Cypress` (macOS) or `~/.cache/Cypress` (Linux) — Cypress binaries
- `du -sh "$LOCALAPPDATA/Cypress/Cache"` — Cypress binaries (Windows, Git Bash)
- `du -sh ~/.cache/playwright` — Playwright browser binaries (macOS/Linux)
- `du -sh "$LOCALAPPDATA/ms-playwright"` — Playwright browser binaries (Windows, Git Bash)
- `du -sh "$LOCALAPPDATA/Yarn/Cache"` — Yarn v1 cache (Windows, Git Bash)
- `du -sh ~/.cache/puppeteer` — Puppeteer Chromium (if exists)
- `find ~ -name node_modules -maxdepth 5 -type d 2>/dev/null | head -30` — locate node_modules dirs

## Analysis

**Caches (Yarn/npm/pnpm):**
- All Safe — re-downloaded on next install, no user data
- pnpm store is shared across projects; clearing it forces re-download for ALL projects, not just one
- Note: Yarn Berry's cache is immutable and deduplicated — it's usually more efficient to keep it

**Browser test caches (Cypress/Playwright/Puppeteer):**
- Safe — re-downloaded automatically before next test run
- These are full Chromium/Electron binaries; typically 300 MB–1.5 GB each

**node_modules:**
- Review — safe to delete (restored with `npm install`/`yarn install`), but:
  - Deleting node_modules in an active project means a re-install before next dev session
  - Only recommend for projects not touched recently (30+ days)
  - Never delete automatically; always list found paths and ask

## Cleanup

Safe   | Clear Yarn v1 cache (macOS) | `rm -rf ~/Library/Caches/Yarn`
Safe   | Clear Yarn v1 cache (Linux) | `rm -rf ~/.yarn/cache`
Safe   | Clear Yarn v1 cache (Windows, Git Bash) | `rm -rf "$LOCALAPPDATA/Yarn/Cache"`
Safe   | Clear npm cache (macOS/Linux) | `npm cache clean --force` or `rm -rf ~/.npm/_cacache`
Safe   | Clear npm cache (Windows, Git Bash) | `npm cache clean --force` or `rm -rf "$APPDATA/npm-cache"`
Safe   | Clear pnpm store (Windows, Git Bash) | `rm -rf "$LOCALAPPDATA/pnpm/store"`
Safe   | Clear Cypress cache (macOS) | `rm -rf ~/Library/Caches/Cypress`
Safe   | Clear Cypress cache (Linux) | `rm -rf ~/.cache/Cypress`
Safe   | Clear Cypress cache (Windows, Git Bash) | `rm -rf "$LOCALAPPDATA/Cypress/Cache"`
Safe   | Clear Playwright cache (macOS/Linux) | `rm -rf ~/.cache/playwright`
Safe   | Clear Playwright cache (Windows, Git Bash) | `rm -rf "$LOCALAPPDATA/ms-playwright"`
Safe   | Clear Puppeteer cache | `rm -rf ~/.cache/puppeteer`
Review | Delete node_modules in a project | `rm -rf <project_path>/node_modules`

## NEVER Delete

- `.yarn/releases/` — Yarn Berry binary (the package manager itself)
- `.yarn/plugins/` — Yarn plugins
- `package.json`, `yarn.lock`, `package-lock.json`, `pnpm-lock.yaml` — lockfiles
