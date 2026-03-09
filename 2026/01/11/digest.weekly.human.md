# Git Mailing List Digest - 2026/01/05 -- 2026/01/11

**The week in brief.** A busy week with 540 emails across 181 threads saw significant progress on multiple fronts. Key developments included the completion of Adrian Ratiu's multi-year hook subsystem standardization (13 patches), Harald Nordgren's 22-iteration journey to add push tracking to `git status`, and Patrick Steinhardt's ongoing refactoring of Git's object database layer. The week also saw Windows symlink support reach final approval and several documentation standardization efforts land. While technical discussions remained focused and productive, some tensions emerged around Rust integration and security defaults.

## Key developments

### Hook subsystem standardization completes

Adrian Ratiu's multi-year effort to standardize Git's hook infrastructure reached its conclusion this week with the final approval of his 13-part v6 series. The changes introduce structured stdin handling, output capture, and parallel execution capabilities while maintaining backward compatibility. All major hooks (post-rewrite, pre-push, reference-transaction, and receive-pack) now use the new `hook.h` API, laying groundwork for future config-based hooks. Junio Hamano's approval ("I didn't see anything iffy") and Patrick Steinhardt's sign-off marked the end of this architectural improvement, which is ready for integration into `next`.

### `git status` push tracking lands after 22 iterations

Harald Nordgren's marathon effort to show push tracking divergence in `git status` output concluded this week with the 22nd iteration receiving final approval. The implementation cleanly extends status output to show both upstream and push tracking when they differ, with comprehensive test coverage (262 lines) verifying edge cases and custom refspec mappings. The change required careful refactoring of branch comparison formatting logic in `remote.c` to maintain existing behavior while adding the new functionality. Junio Hamano and Phillip Wood provided extensive review throughout the series' evolution, which demonstrates Git's rigorous review process in action.

### Windows symlink support finalized

Johannes Schindelin and Karsten Blees' 18-patch series implementing comprehensive POSIX-like symlink support on Windows received final approval this week. The changes touch core MinGW compatibility code, addressing everything from symlink creation/reading to directory handling and error cases. This represents the culmination of work that began in Git for Windows in 2015, with particular attention to edge cases like permission handling and index corruption. The series includes proper attribution and documentation accuracy, marking a significant milestone in Windows compatibility.

### ODB abstraction advances

Patrick Steinhardt's object database refactoring work saw significant progress this week with multiple series moving forward. His 7-patch v4 series improving object info handling demonstrated 9% speedups in disk usage calculations, while the larger packfile store relocation effort (moving MIDX tracking to odb_source) continued review. These changes are part of GitLab's ongoing effort to enable pluggable ODB backends. The work exemplifies Steinhardt's characteristic approach: careful, incremental refactoring with thorough test coverage and clear architectural boundaries.

### Submodule gitdir path encoding ready

Adrian Ratiu's 12-patch series implementing configurable submodule gitdir paths via `extensions.submodulePathConfig` reached its eighth and final iteration this week. The feature allows custom gitdir locations while preventing filesystem conflicts through a four-tier fallback system (plain/encoded/numbered/hashed paths) with RFC3986 encoding for case-folding protection. Josh Steadmon confirmed successful testing against Google's downstream implementation, clearing the last hurdle before merging. The series includes comprehensive test coverage and an atomic migration command (`git submodule--helper migrate-gitdir-configs`) to convert existing repositories.

## In brief

**`git-history` command refinements** -- Patrick Steinhardt's experimental `git-history` command saw significant review from Elijah Newren, with v9 addressing error handling and documentation polish for the initial `reword` subcommand.

**Reftable improvements** -- Tsahi Elkayam corrected undefined behavior in the indexed table reference iterator and proposed optimizations for object-to-ref lookups in Patrick Steinhardt and Karthik Nayak's reftable backend.

**Documentation standardization** -- Michael Lyons completed the `git-blame` and `git-bisect` man page conversions to synopsis-style format, part of Jean-Noël Avila's ongoing effort.

**`git reset` documentation** -- D. Ben Knoble shepherded the final iteration of Julia Evans' pedagogical improvements to the `git reset` man page, now with consistent formatting and imperative mood.

**Test modernization** -- Multiple contributors replaced direct shell tests with helper functions in t1300, t1420, t2021, and t5403, improving failure diagnostics and maintainability.

**MacOS iconv workarounds** -- Torsten Bögershausen and René Scharfe proposed platform-specific fixes for macOS 14/15's buggy iconv implementation affecting ISO-2022-JP conversions.

**`git fsck` race condition** -- Elijah Newren, Jeff King, and Junio Hamano refined a bugfix addressing race conditions in `git fsck` when run on live repositories.

**`git add -p` UI improvements** -- A GSoC participant's patch to show previous hunk decisions during interactive staging is ready after 7 iterations, providing clearer feedback.

**PID file debugging** -- Paulo Casaretto's feature to help diagnose stale locks via `~pid.lock` files is now production-ready with Windows support and automatic cleanup.

**French translation update** -- Jean-Noël Avila brought the French `.po` file up to date with current source strings.

## Looking ahead

**Rustification tensions** -- brian m. carlson's comments about NULL pointer handling in the hook subsystem thread highlighted ongoing debates about language safety as Ezekiel Newren's Rust integration work continues.

**`the_repository` removal** -- Outreachy intern Bello Olamide's report on architectural challenges around config variable deglobalization suggests this multi-year effort will see continued activity.

**Geometric repacking** -- Patrick Steinhardt's series addressing promisor pack handling in geometric repacking received initial review from Taylor Blau, with test case design still under discussion.

**Bazel build support** -- After technical objections, Son Luong Ngoc has shifted to an out-of-tree solution via Bazel Central Registry rather than in-tree support.

**New contributor onboarding** -- Lorenzo Pegorari's introduction as a prospective GSoC 2026 participant may lead to new activity in documentation or testing areas.