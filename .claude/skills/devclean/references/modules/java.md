# Java (Gradle + Maven)

platform: all
detect: which java || test -d ~/.gradle || test -d ~/.m2

## Scan

- `du -sh ~/.gradle/caches` — Gradle dependency cache (if exists)
- `du -sh ~/.gradle/wrapper/dists` — Gradle wrapper distributions (if exists)
- `du -sh ~/.m2/repository` — Maven local repository (if exists)
- `find ~ -name ".gradle" -maxdepth 5 -type d 2>/dev/null | xargs du -sh 2>/dev/null | sort -rh | head -10` — per-project Gradle dirs
- `find ~ -name "build" -maxdepth 5 -type d 2>/dev/null | xargs -I{} sh -c 'test -f "{}/../build.gradle" || test -f "{}/../build.gradle.kts" && du -sh "{}"' 2>/dev/null | sort -rh | head -10`

## Analysis

**Gradle dependency cache (`~/.gradle/caches`):**
- Safe — re-downloaded on next build; Gradle manages its own cache invalidation
- Often 2–10 GB on active Android/Java projects
- Use `gradle cleanBuildCache` for project-specific build caches

**Gradle wrapper dists (`~/.gradle/wrapper/dists`):**
- Review — each Gradle version takes ~100–200 MB
- Versions no longer referenced in any project's `gradle-wrapper.properties` are orphaned
- Check `grep -r "distributionUrl" ~/projects/**/gradle-wrapper.properties 2>/dev/null` for in-use versions

**Maven local repository (`~/.m2/repository`):**
- Review — contains all downloaded Maven artifacts; safe to delete but forces a full re-download for every Maven project on this machine
- Typically 5–20 GB; triggers a rebuild cost proportional to the number of active projects — recommend only when space is critical or artifacts are stale

**Project build directories:**
- Review — same as Rust `target/`; safe to delete, full rebuild required

## Cleanup

Safe   | Clear Gradle caches | `rm -rf ~/.gradle/caches`
Review | Remove old Gradle wrapper versions | `rm -rf ~/.gradle/wrapper/dists/<old_version>`
Review | Clear Maven local repo | `rm -rf ~/.m2/repository`
Review | Clean project build dir (Gradle) | `./gradlew clean` or `rm -rf <project>/build`
Review | Clean project build dir (Maven) | `mvn clean` or `rm -rf <project>/target`

## Windows Notes

All standard paths resolve correctly in Git Bash on Windows via `$USERPROFILE`:
- Gradle cache: `$USERPROFILE/.gradle/caches` (`%USERPROFILE%\.gradle\caches`)
- Gradle wrapper dists: `$USERPROFILE/.gradle/wrapper/dists` (`%USERPROFILE%\.gradle\wrapper\dists`)
- Maven repository: `$USERPROFILE/.m2/repository` (`%USERPROFILE%\.m2\repository`)

No path changes are needed for Scan or Cleanup commands when using Git Bash — `~` resolves to `$USERPROFILE` on Windows.

## NEVER Delete

- `~/.gradle/gradle.properties` — global Gradle config (may contain API keys, JVM flags)
- `~/.m2/settings.xml` — Maven settings (may contain repo credentials)
- `<project>/gradle/wrapper/gradle-wrapper.jar` — the wrapper bootstrap binary
