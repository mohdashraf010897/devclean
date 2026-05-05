# Rust / Cargo

platform: all
detect: which cargo || test -d ~/.cargo

## Scan

- `du -sh ~/.cargo/registry/cache` — downloaded crate tarballs
- `du -sh ~/.cargo/registry/src` — extracted crate sources
- `du -sh ~/.cargo/git/db` — git-sourced crate clones
- `du -sh ~/.cargo/git/checkouts` — git crate checkouts
- `find ~ -name "target" -maxdepth 5 -type d 2>/dev/null | xargs -I{} sh -c 'test -f "{}/../Cargo.toml" && du -sh "{}"' | sort -rh | head -15`

## Analysis

**Cargo registry cache and src:**
- Safe — `cargo` re-downloads and re-extracts on next build if missing
- `registry/src` is larger than `registry/cache`; both are safe
- Use `cargo cache --autoclean` (if `cargo-cache` is installed) for smarter cleanup

**Git dependency cache:**
- Safe — re-cloned on next build

**`target/` directories:**
- Review — these are build artifacts (compiled objects, binaries, test artifacts)
- Safe to delete: Cargo rebuilds from source on next `cargo build`
- Cost: full recompile of the project and all deps (can be 5–30 min for large projects)
- Recommend cleaning `target/` for projects inactive 30+ days
- `target/` can be 1–10 GB per project depending on deps and debug vs release builds
- Incremental compilation cache inside `target/` is separate from the output binary

## Cleanup

Safe   | Clear crate download cache | `rm -rf ~/.cargo/registry/cache`
Safe   | Clear crate source extracts | `rm -rf ~/.cargo/registry/src`
Safe   | Clear git dependency clones | `rm -rf ~/.cargo/git`
Review | Clean a project's build artifacts | `cargo clean` (run inside project) or `rm -rf <project>/target`
Safe   | Smart Cargo cache cleanup (if cargo-cache installed) | `cargo cache --autoclean`

## Windows Notes

- `~/.cargo/` maps to `%USERPROFILE%\.cargo\` on Windows; in Git Bash, `$USERPROFILE` equals `~`, so all paths above work unchanged
- `target\` directories use backslashes in Windows Explorer, but Git Bash `find` works correctly with forward slashes
- `cargo clean` and all `rm -rf ~/.cargo/...` commands work the same in Git Bash on Windows
- No path changes are needed for any Scan or Cleanup commands when using Git Bash

## NEVER Delete

- `~/.cargo/bin/` — installed Cargo binaries (`cargo`, `rustfmt`, `clippy`, user-installed tools)
- `~/.cargo/env` — Cargo environment setup
- `~/.rustup/` — Rust toolchain installation (entire Rust install lives here)
- `Cargo.toml`, `Cargo.lock` — project manifests and lockfiles
