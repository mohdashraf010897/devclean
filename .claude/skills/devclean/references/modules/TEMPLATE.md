# Module Name

platform: macos|linux|windows|all
detect: test -d /path/that/proves/this/tool/is/installed

## Scan

<!-- List du/find commands to measure this toolchain's footprint -->
<!-- Use (if exists) notation for optional paths -->

## Analysis

<!-- How to classify findings. Must use: Safe / Review / Risky -->
<!-- Include any config-reading or cross-referencing logic -->

## Cleanup

<!-- Exact shell commands, one per cleanup action -->
<!-- Format: Risk | Description | Command -->

## NEVER Delete

<!-- Tool-specific paths that must never be removed -->
<!-- These extend the universal list in references/REFERENCE.md -->
