# Git Mailing List Digest - 2026/01/05 -- 2026/01/11

**The week in brief.** A busy week with 544 emails across 181 threads saw significant progress on multiple fronts. The hook subsystem standardization and submodule gitdir path encoding efforts reached completion after extensive review, while `git status` push tracking and Windows symlink support were finalized after 22 and 5 iterations respectively. Patrick Steinhardt advanced several major initiatives including ODB abstraction, geometric repacking with promisor remotes, and the new `git-history` command. Documentation work continued steadily with synopsis-style conversions and test modernization efforts.

## Key developments

**Hook subsystem standardization completes**  
Adrian Ratiu's multi-year effort to standardize Git's hook infrastructure has cleared all administrative hurdles. The 13-part v6 series introduces structured stdin handling, output capture, and parallel execution capabilities while maintaining backward compatibility. All major hooks (post-rewrite, pre-push, reference-transaction, and receive-pack) now use the new `hook.h` API, laying groundwork for future config-based hooks. With explicit approval from Junio Hamano and Patrick Steinhardt, plus passing CI, this architectural improvement is now ready for integration into `next`.

**Submodule gitdir path encoding finalized**  
After 8 iterations and extensive review, Adrian Ratiu's series implementing configurable submodule gitdir paths via `extensions.submodulePathConfig` received final approval. The feature allows custom gitdir locations while preventing filesystem conflicts through a four-tier fallback system with RFC3986 encoding for case-folding protection. The implementation includes comprehensive test coverage and an atomic migration command (`git submodule--helper migrate-gitdir-configs`) to convert existing repositories. Josh Steadmon confirmed successful testing against Google's downstream implementation, clearing the last hurdle before merging.

**`git status` push tracking enhancement lands**  
Harald Nordgren's 22-iteration series adding push remote tracking information to `git status` output was finalized this week. The implementation shows divergence from both upstream and push tracking branches when they differ, with 262 lines of test coverage verifying edge cases and custom refspec mappings. Junio Hamano and Phillip Wood provided extensive review, with the final version addressing a subtle control flow regression in the refactoring that could incorrectly report divergence. The change required careful refactoring of branch comparison formatting logic in `remote.c` to maintain existing behavior while adding the new functionality.

**Windows symlink support ready**  
Johannes Schindelin and Karsten Blees' 5-patch series implementing comprehensive POSIX-like symlink support on Windows received final approval. The changes touch core MinGW compatibility code, addressing everything from symlink creation/reading to directory handling and error cases. The series represents the culmination of work that began in Git for Windows in 2015, with particular attention to edge cases like permission handling and index corruption. All technical and administrative issues are resolved, including proper attribution and documentation accuracy.

**ODB abstraction advances**  
Patrick Steinhardt's object database refactoring work saw significant progress with multiple series moving forward. His 7-patch v4 series improving object info handling demonstrated 9% speedups in disk usage calculations. Meanwhile, the larger packfile store relocation effort (moving MIDX tracking to odb_source) continued review with Toon Claes confirming the architectural soundness. These changes are part of GitLab's ongoing effort to enable pluggable ODB backends, with steady progress throughout the week.

## In brief

**Reftable iterator fix** -- Tsahi Elkayam corrected undefined behavior in the indexed table reference iterator, with Patrick Steinhardt noting opportunities for future optimization in the object index design.

**French translation update** -- Jean-Noël Avila brought the French `.po` file up to date with current source strings.

**Test modernization** -- Multiple contributors replaced direct shell tests with helper functions across t1300, t1420, t2021, t5403, and t1410, improving failure diagnostics.

**Shallow clone ref-in-want fix** -- Matthew Dodd identified and fixed a protocol violation where upload-pack sent wanted-refs before shallow-info when `uploadpack.allowRefInWant` was enabled.

**`git subtree` regression analysis** -- Colin Stagner's fix for squash commit handling not only corrected functionality but improved performance by 3x in some cases.

**Linux fsmonitor meson support** -- Patrick Steinhardt added build configuration for the new Linux inotify backend to meson.build.

**`git add -p` initialization fix** -- SZEDER Gábor caught an uninitialized variable in the recently merged hunk decision display feature.

**PID tracking for lock files** -- Paulo Casaretto's feature to help diagnose stale locks via `~pid.lock` files is now production-ready with Windows support and automatic cleanup.

**MacOS iconv workarounds** -- Torsten Bögershausen and René Scharfe proposed platform-specific fixes for macOS 14/15's buggy iconv implementation affecting ISO-2022-JP conversions.

**Documentation standardization** -- Michael Lyons completed `git-blame` and `git-bisect` man page conversions to synopsis-style format as part of Jean-Noël Avila's ongoing effort.

**`git fsck` race condition fix** -- Elijah Newren, Jeff King, and Junio Hamano refined a comprehensive fix addressing race conditions when running `git fsck` on live repositories.

## Looking ahead

**`git-history` command** -- Patrick Steinhardt's series is nearing readiness after addressing Elijah Newren's review feedback, with some forward compatibility considerations remaining for future subcommands.

**`the_repository` removal** -- Phillip Wood highlighted user experience regressions from lazy config parsing during migration, suggesting this architectural effort will require careful coordination.

**Rustification tensions** -- brian m. carlson's comments about NULL pointer handling in the hook subsystem thread highlight ongoing debates about language safety as Rust integration work continues.

**Geometric repacking with promisors** -- Patrick Steinhardt's series addressing promisor pack handling received initial review from Taylor Blau, with test case design discussions ongoing.

**Bazel build support** -- After technical objections, Son Luong Ngoc has shifted to pursuing an out-of-tree solution via Bazel Central Registry rather than direct integration.