# Git Mailing List Digest — 2025/03/05

## The day in brief

A moderately active day with 41 emails across 15 threads, featuring the culmination of Karthik Nayak's partial reference transaction series, continued discussion of cruft pack handling, and the Git for Windows 2.49.0-rc1 release announcement. The most notable development was the v3 submission of Karthik's long-running partial reference transaction implementation, now with optimized error reporting and comprehensive backend support.

## Notable threads

### Partial reference transactions reach final form

Karthik Nayak submitted the final version (v3) of his 8-patch series introducing partial reference transaction support, allowing individual reference updates to fail while others proceed when using `git update-ref --allow-partial`. The implementation now includes:

- A new `ref_transaction_rejections` struct for efficient error tracking
- Consolidated error handling via `ref_transaction_maybe_set_rejected()`
- Support for partial failures during F/D conflict checks
- Comprehensive test coverage (files, packed, reftable backends)

Junio Hamano raised a philosophical question about whether "partial transactions" is the right framing, suggesting "non-transactional batched updates" might better describe the behavior. The technical implementation appears settled after extensive review rounds, with the discussion now focusing on conceptual clarity.

### Cruft pack freshening fix finalized

Taylor Blau submitted the final version of a bugfix addressing object freshening behavior with multiple cruft packs. The patch modifies `want_found_object()` in `pack-objects.c` to properly handle mtime comparisons when freshening objects from retained cruft packs. Earlier versions had included a problematic simplification of cruft pack aggregation logic that was withdrawn after Patrick Steinhardt identified potential repacking cycles. The now-simplified series focuses solely on the freshening fix with comprehensive test coverage in `t7704-repack-cruft.sh`.

### Git for Windows 2.49.0-rc1 released

Johannes Schindelin announced Git for Windows 2.49.0-rc1, featuring several platform-specific changes:

- Planned deprecation of `git svn` due to maintenance burden
- Discontinuation of 32-bit installers (except MinGit until 2029)
- Fixes for Windows-specific issues including symlink handling and VS Code terminal hangs
- Incorporation of upstream changes like stabilized `--name-hash-version=2`

The release includes updated dependencies (OpenSSH 9.9.P2, PCRE2 10.45) and checksums for various distribution formats.

## In brief

Jeff King and Taylor Blau continued discussing `transfer.hideRefs` behavior inconsistencies across ref backends, with Taylor proposing solutions to make packed refs matching consistent with loose refs. David Mandelberg submitted v2 of his bash completion fixes for remote names containing slashes, now using a path-counting helper to reduce forks. Phillip Wood landed documentation changes for build-time conditional documentation of deprecated features using new AsciiDoc attributes. The meson build system thread saw interpersonal reassurance after technical resolution. The mentoring program subject prefix discussion concluded with documentation updates to recommend single-bracket format ("[GSoC PATCH]"). 

## On the radar

The language-specific `.gitignore` proposal saw continued discussion with maintainer pushback against expanding core Git's responsibilities, though alternative solutions using the existing template system remain under consideration. Ayush Chandekar began exploring GSOC work on `the_repository` removal, receiving guidance about focusing on simpler global states first. The `hideRefs` inconsistency fix appears headed toward a solution that would move common logic to the generic refs layer.