Here's the daily digest for November 17, 2025:

## The day in brief

November 17 saw significant activity with 84 emails across 35 threads, featuring major developments in Rust integration, the release of Git v2.52.0, and several feature series nearing completion. The day was brian m. carlson's v2 series introducing Rust infrastructure for SHA-1/SHA-256 interoperability, while Junio Hamano's "What's cooking" report and release announcement provided key project milestones.

## Notable threads

### Rust infrastructure for hash algorithm interoperability

brian m. carlson submitted v2 series (15 patches) introducing Rust components for SHA-1/SHA-256 interoperability. The series establishes foundational Rust infrastructure including:
- `ObjectID` struct mirroring C's `object_id`
- `HashAlgorithm` enum with FFI-safe operations
- New binary object map format in `src/loose.rs`
- Comprehensive test coverage and build system updates

The implementation maintains strict FFI compatibility while providing type safety on the Rust side. A Windows/Meson build issue was noted but appears pre-existing. This represents Git's first major Rust integration, with careful attention to cross-language type safety and performance characteristics.

### Git v2.52.0 released

Junio Hamano announced Git v2.52.0, featuring:
- New commands: `git refs`, `git repo`, `git last-modified`
- IMAP folder support in `git send-email`
- Default branch name warnings for Git 3.0 transition
- Continued `the_repository` removal and Rust integration
- Geometric repacking strategy for maintenance

The release includes 637 non-merge commits from 94 contributors (33 new), marking a significant milestone in Git's evolution while maintaining stability.

### `git blame` gains diff algorithm configurability

Antonin Delpeuch's series adding `--diff-algorithm` to `git blame` received final approval after six iterations. The implementation:
- Adds support for myers/minimal/patience/histogram algorithms
- Respects both CLI option and `diff.algorithm` config
- Includes comprehensive tests in t8015
- Maintains backward compatibility with `--minimal`

The feature is now cleared for merging after addressing all review feedback, including final documentation and test refinements.

### `git fast-import` signature validation

Christian Couder's v2 series adds `--strip-if-invalid` mode to `git fast-import`'s `--signed-commits`. The implementation:
- Validates signatures using `git verify-commit` logic
- Strips invalid signatures with warning messages
- Handles OpenPGP/X.509/SSH signature types
- Includes extensive test coverage

The series addresses `git filter-repo` use cases where users want to rewrite history while preserving valid signatures. Documentation now clearly states the validation matches `git verify-commit`.

### `git-repo-info` nears completion

The experimental `git-repo-info` feature series reached final review after addressing all feedback in v4. Junio Hamano noted minor documentation improvements for future consideration but confirmed the implementation is ready. The command provides repository metadata querying with multiple output formats and now includes an `--all` flag to display all available fields.

## In brief

**Git for Windows credential helper fix** -- David Macek and Johannes Schindelin fixed memory corruption in the wincred helper where `wcsncpy_s()` wasn't allocating space for null termination.

**CMake build system sync** -- Johannes Schindelin removed obsolete CMake logic for xdiff and reftable libraries after their consolidation into libgit in the Makefile build.

**Windows clang warning fixes** -- Johannes Schindelin addressed `-Wcomma` warnings in mingw.c by separating `errno` assignments from return statements.

**Submodule hash algorithm edge case** -- Martin Wilck suggested adding warnings for rare SHA-1 submodule OIDs that coincidentally end in 24 zeros, though the core validation approach remains unchanged.

**Makefile strip target cleanup** -- Junio Hamano proposed restructuring the `make strip` target to use `$(OTHER_PROGRAMS)` rather than explicit binary lists, following Johannes Schindelin's Scalar inclusion patch.

## On the radar

**`git fetch` tag handling regression** -- Karthik Nayak and Junio Hamano identified additional non-atomic fetch behaviors needing fixes, though the core tag-fetch solution will merge first.

**Rust Windows/Meson build issue** -- A CI failure in the SHA-1/SHA-256 interoperability series appears pre-existing but may need platform-specific expertise to resolve.

**Worktree documentation debate** -- Johannes Schindelin provided real-world evidence that nested worktrees can prevent data loss, potentially reversing the documentation's current recommendation against nesting.