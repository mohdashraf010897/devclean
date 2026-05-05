# Python

platform: all
detect: which python3 || which python || which pip3

## Scan

- `du -sh ~/.cache/pip` (Linux/macOS) — pip cache
- `du -sh ~/Library/Caches/pip` (macOS alternative)
- `du -sh "$LOCALAPPDATA/pip/Cache"` (Windows, Git Bash) — pip cache
- `du -sh ~/.pyenv/cache` — pyenv downloaded Python tarballs (if exists, macOS/Linux)
- `du -sh "$USERPROFILE/.pyenv/pyenv-win/cache"` — pyenv-win tarballs (Windows, Git Bash)
- `du -sh ~/.conda/pkgs` — Conda package cache (if exists)
- `du -sh ~/anaconda3/pkgs` or `~/miniconda3/pkgs` — Anaconda/Miniconda cache (macOS/Linux)
- `du -sh "$USERPROFILE/Anaconda3/pkgs"` or `du -sh /c/ProgramData/Anaconda3/pkgs` — Anaconda cache (Windows, Git Bash)
- `find ~ -name "__pycache__" -maxdepth 6 -type d 2>/dev/null | head -20` — bytecode caches
- `find ~ -name "*.pyc" -maxdepth 6 2>/dev/null | wc -l` — count stale .pyc files
- `find ~ -name ".venv" -o -name "venv" -o -name "env" -maxdepth 5 -type d 2>/dev/null | head -20` — virtual environments (same directory names on Windows; Git Bash `find` handles forward slashes)

## Analysis

**pip cache:**
- Safe — re-downloaded on next `pip install`

**pyenv tarball cache:**
- Safe — these are the source tarballs used to build Python versions; once installed, the version is in `~/.pyenv/versions/` and the tarball isn't needed

**Conda/Anaconda package cache:**
- Safe to clear with `conda clean --all --yes`; packages re-downloaded on next env solve

**__pycache__ and .pyc files:**
- Safe — regenerated automatically by Python on next import
- Note: `.pyc` files in system packages (`/usr/lib/python*/`) require root and should be skipped

**Virtual environments (.venv / venv / env):**
- Review — safe to delete if the project can be reinstalled (`pip install -r requirements.txt`)
- Only recommend for projects inactive 30+ days
- Identify by parent directory and last modified time

## Cleanup

Safe   | Clear pip cache (Linux/macOS) | `pip cache purge` or `rm -rf ~/.cache/pip`
Safe   | Clear pip cache (Windows, Git Bash) | `pip cache purge` or `rm -rf "$LOCALAPPDATA/pip/Cache"`
Safe   | Clear pyenv tarball cache (macOS/Linux) | `rm -rf ~/.pyenv/cache`
Safe   | Clear pyenv-win tarball cache (Windows, Git Bash) | `rm -rf "$USERPROFILE/.pyenv/pyenv-win/cache"`
Safe   | Clear Conda package cache | `conda clean --all --yes`
Safe   | Remove __pycache__ directories | `find <project> -type d -name __pycache__ -exec rm -rf {} +`
Safe   | Remove .pyc files | `find <project> -name "*.pyc" -delete`
Review | Delete a virtual environment | `rm -rf <project>/.venv`

## NEVER Delete

- `~/.pyenv/versions/` — installed Python interpreters
- `~/.pyenv/shims/` — pyenv shims
- `~/anaconda3/envs/` or `~/miniconda3/envs/` — named Conda environments (user data)
- `requirements.txt`, `Pipfile`, `pyproject.toml` — dependency declarations
