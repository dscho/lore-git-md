# Git Mailing List Digest - 2026/04/21 (Tuesday)

**The day in brief.** A busy Tuesday with 89 emails across 23 threads, dominated by Taylor Blau's major MIDX repacking series and Patrick Steinhardt's test suite modernization. Key highlights include the completion of Taylor's incremental MIDX repacking architecture and Patrick's `set -e` test suite changes being marked ready for merge. Several other important threads saw significant progress, including subtree performance discussions and security fixes.

## Notable threads

### Incremental MIDX repacking series finalized

Taylor Blau's **16-patch series introducing incremental MIDX/bitmap-based repacking** reached completion today, representing a major architectural improvement for large repositories. The final version implements two operational modes:

1. **Geometric MIDX chain repacking** (recommended path) via `--write-midx=incremental --geometric`, creating a structure where older layers contain fewer, larger packs while newer layers contain many smaller packs
2. **Append-only MIDX chain growth** via standalone `--write-midx=incremental`, where each repack adds a new layer containing just the newly written packs

The series includes comprehensive test coverage and addresses all review feedback from Elijah Newren and others. Performance validation shows significant reductions in repacking overhead for large monorepos that repack frequently. This completes a multi-year effort Taylor has been presenting at conferences, with the implementation now ready for final review and integration.

### Test suite modernization with `set -e`

Patrick Steinhardt's **12-patch series to make Git's test suite compatible with `set -e` (errexit)** was marked ready for merging after six iterations and thorough review. The changes:

- Make `set -e` the default for Bash 5+ (with opt-in via `GIT_TEST_USE_SET_E` for other shells)
- Include CI configuration to enable strict error checking for Linux jobs
- Adapt test patterns across the suite to handle immediate failure on errors
- Add careful boolean parsing of environment variables to avoid edge cases

The work significantly improves test reliability by ensuring failures outside test cases surface immediately rather than being silently ignored. Jeff King (Peff) confirmed the final technical details, and the series has addressed all substantive feedback. This represents a major step forward in Git's ongoing effort to modernize and harden its test infrastructure.

### Subtree performance discussion shifts to deployment strategy

The long-running `contrib/subtree` performance discussion transitioned from technical implementation choices to **deployment strategy for Colin Stagner's shell patches versus Ian Jackson's Rust/libgit2 version**. Colin provided concrete guidance on test coverage and naming strategies to enable parallel development paths:

- References existing test suite in `git-subtree.sh` as compatibility baseline
- Suggests distinct naming (`git-subtree2` or similar) for Rust version
- Proposes optional symlink/alias mechanism for transitional deployment

The thread has moved from contention to coordinated separation of solutions, with Junio Hamano's policy against Rust in git.git remaining the binding constraint. The outcome establishes a pattern for other components facing similar modernization challenges when core constraints prevent in-tree adoption.

### Commit signing regression fixed

brian m. carlson addressed a **commit signing regression for non-UTF-8 messages** that had been present since v2.45.0. The issue caused signature mismatches when messages contained non-UTF-8 bytes without explicit encoding configuration. The fix:

- Renames `verify_utf8()` to `ensure_utf8()` to better describe its behavior
- Moves UTF-8 validation/transcoding before signing operations
- Ensures signed content matches stored content
- Adds test case using invalid UTF-8 bytes (0xfe and 0xff) for regression protection

Kushal Das confirmed the fix works for real-world use cases while suggesting expanded test coverage for non-GnuPG signing tools. The solution restores pre-2.45.0 behavior where signatures remain valid for non-UTF-8 messages.

### `the_repository` removal progresses in setup subsystem

Patrick Steinhardt's **18-patch series completing the `the_repository` removal in the setup subsystem** saw substantive discussion today. Key points addressed:

- Performance impact analysis for `initialized` flag removal in `setup_work_tree()`, showing bounded overhead
- Parameter passing consistency in `repo_migrate_ref_storage_format()`
- General approach to large-scale refactorings: focusing on subsystem conversions while minimizing caller changes

The series converts functions in setup.c and their callers to use explicit `struct repository *` parameters instead of the global variable. With 116 files changed (560+/552-), this represents significant progress in the long-running effort to eliminate `the_repository`.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Git-gui bare repo handling** -- Improved error messages and fallback behavior when launched from bare repositories or inaccessible worktrees.

**Ninja build system fix** -- Compatibility patch for older Ninja versions (pre-1.10.0) that incorrectly handle multi-line dependencies in config-list.h generation.

**Git Merge 2026 announcement** -- Conference scheduled for September 17-18 in Lisbon, with Contributor's Summit and Jujutsu (jj) pre-event.

**Pseudo-merge bitmap fixes** -- Taylor Blau's v2 series addresses multiple issues in pseudo-merge bitmap implementation with comprehensive test coverage.

**Test modernization for t7004-tag.sh** -- Siddharth Shrimali's series replaces brittle test patterns with robust verification methods, now approved by Patrick Steinhardt.

**Merge-ort hardening** -- Elijah Newren's series fixes duplicate tree entry handling and improves directory/file conflict detection in merge-ort.

**Git grep` column number bug** -- René Scharfe confirms and analyzes incorrect column reporting with `--only-matching`, proposing a fix to match GNU grep's behavior.

## On the radar

**Source tree reorganization** -- Patrick Steinhardt's proposal to move libgit.a components into a "lib/" directory gains consensus on incremental approach, with execution planning underway.

**Credential helper security** -- `git-credential-libsecret` found leaking credentials to stdout during store operations, with discussion ongoing about interface boundaries.

**Negotiation control series** -- Derrick Stolee's fetch/push negotiation improvements are in final polishing phase with only naming cleanup remaining before integration.