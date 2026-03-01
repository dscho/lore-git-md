# Git Mailing List Digest - 2025/01/30

**The day in brief.** A busy Thursday with 87 emails across 21 threads, featuring significant progress on multiple fronts. Key developments include the approval of Rust bindings for Git's C library, ongoing security-focused work on promisor remotes, and multiple bugfix series nearing completion. The day also saw continued discussion around GSoC 2025 planning and build system improvements.

## Notable threads

### Rust bindings approved for integration

The foundational work for Rust integration with Git's C library reached a milestone today with Phillip Wood's positive review and Junio Hamano's approval of Josh Steadmon's Rust bindings series (v9). This series introduces two Rust crates in `contrib/` - `libgit-sys` for FFI bindings and `libgit` for higher-level Rust interfaces - implementing proper memory management and build system integration. The implementation has been validated through real-world use in the JJ VCS and is now cleared for merging into 'next', marking an important step in Git's gradual Rust adoption.

### Promisor remote security architecture evolves

Patrick Steinhardt proposed a significant architectural change to how promisor remotes are identified and managed, suggesting replacement of name-based identification with an opaque ID system (like UUIDs) to address security concerns. This would introduce three granular configuration controls ("new", "update", "prune") and modify the protocol v2 capability to advertise IDs rather than names. The proposal comes alongside detailed technical reviews of Christian Couder's promisor-remote capability implementation, where Patrick identified security gaps in remote name validation and URL handling.

### Packed-refs validation series progresses

Sheji Luo's packed-refs validation series (v2) saw extensive review today, with Junio Hamano providing detailed feedback on multiple patches. The series implements strict validation of packed-refs file structure while deferring object validation to future work. Key discussion points included header format validation (where Junio noted the need to maintain backward compatibility with pre-v1.5.0 headerless files) and sortedness verification (which should only apply when explicitly declared in the header). The series demonstrates Git's careful approach to validation - ensuring thorough checking while respecting historical repository formats.

### Repository reinitialization fixes

Patrick Steinhardt submitted a series addressing repository reinitialization issues when ref or object formats differ from the original configuration. The changes prevent environment variables (`GIT_DEFAULT_REF_FORMAT` and `GIT_DEFAULT_HASH`) from overriding existing repository formats during reinitialization, a problem observed in GitLab's systems. Junio Hamano and brian m. carlson both approved the conservative approach of blocking format changes during reinit rather than attempting migration, though the door remains open for future work in this area.

## In brief

**Memory leaks fixed** - Junio approved Patrick Steinhardt's 2-patch series fixing leaks in Unix socket handling (triggered by long paths) and scalar.c's clone operation, both detected by Meson's stricter leak checking.

**Documentation build conflict resolved** - Consensus was reached on resolving the .txt vs .adoc conflict between Jean-Noël Avila's AsciiDoc conversion and Adam Johnson's doc-fix branch through specific file adjustments.

**GSoC 2025 planning continues** - Discussion evolved around microproject documentation, with Patrick Steinhardt and Christian Couder debating project inclusion criteria and Junio proposing an automatic expiration system for stale" entries.

**Meson build improvements** - Patrick Steinhardt submitted v2 of his Meson refactoring series (13 patches) with Windows-specific fixes and dependency management improvements, including targeted PATH handling and OpenSSL fallback fixes.

**Test modernization** - A 4-patch series converted several unit tests (hashmap, decorate, strbuf, strcmp-offset) to the Clar framework under Patrick Steinhardt's mentorship, maintaining coverage while adopting the new test style.

**On the radar**

The reported regression in tag fetching behavior (bisected to commit 3f763ddf28) warrants monitoring as it affects protocol v2 behavior with depth limits. Igor Todorov's clear reproduction case shows tags missing from `git log --decorate` output in specific fetch scenarios.