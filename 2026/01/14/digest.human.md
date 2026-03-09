Here's the daily digest for January 14, 2026:

## The day in brief

A busy day with 131 emails across 25 threads, featuring significant progress on several major technical efforts. The MIDX compaction series reached final review, the submodule `ignore=all` behavior was finalized, and multiple bugfixes landed for hook execution, HTTP authentication, and error reporting. The day also saw extensive discussion about triangular workflows following the recent push-tracking status feature merge.

## Notable threads

**MIDX compaction with bitmap support finalized**  
Taylor Blau's 18-patch series implementing MIDX compaction functionality with reachability bitmap support reached its final review stages. The series introduces a new `git multi-pack-index compact` subcommand that combines adjacent MIDX layers while preserving bitmap compatibility. Junio Hamano and Patrick Steinhardt provided thorough review feedback, with only minor documentation tweaks remaining before merging. The implementation includes comprehensive test coverage and careful attention to backward compatibility with the new MIDX v2 format.

**Submodule `ignore=all` behavior settled**  
After multiple iterations, Claus Schneider's series aligning `git add` behavior with other commands for submodules configured with `ignore=all` was finalized. The v3 implementation uses `--force` rather than the originally proposed `--include-ignored-submodules` flag name, settling a naming debate from earlier versions. The changes ensure consistent handling across commands while maintaining backward compatibility and adding comprehensive test coverage. Junio approved the technical approach with only minor style fixes remaining.

**Hook execution improvements and fixes**  
Adrian Ratiu's bugfix series addressing hook execution issues saw extensive discussion, particularly around stdin/stderr handling and deadlock scenarios in batched operations. Jeff King identified a serious deadlock risk in the current approach to feeding stdin while reading stderr. The v3 patches split the fixes into clear components: stdout/stderr separation restoration and `ungroup` flag behavior correction. The thread revealed deeper architectural questions about hook execution that may inform future design changes.

**Error reporting in batched reference updates**  
Karthik Nayak and Jeff King completed a series restoring detailed error messages in batched reference operations (`update-ref`, `fetch`, and `receive-pack`). The changes propagate backend-specific error details through the ref transaction infrastructure while maintaining backward compatibility. The series was merged after thorough review, though post-merge discussion continued about optimizing error message formatting and preserving machine-readability.

**Triangular workflow discussions continue**  
Following Harald Nordgren's recently merged push-tracking status feature, the thread explored configuration ergonomics for triangular workflows (where push and pull remotes differ). Junio proposed a new `status.compareBranches` configuration to make the feature optional, while others debated whether push tracking should be enabled by default. The discussion highlighted real-world workflow variations and the tension between discoverability and interface cleanliness.

## In brief

**HTTP authentication fix for large requests** -- Aaron Plattner corrected missing auth headers in `probe_rpc()` calls during large clone operations, with comprehensive test coverage.

**Ref snapshotting for `git fsck` races** -- Elijah Newren's series to handle concurrent repository modifications during fsck operations was approved for 'next' after addressing all review feedback.

**Lisp dialect userdiff finalized** -- Scott Burson and Johannes Sixt completed the unified Lisp syntax highlighting support with fixes for vertical bar symbol handling.

**`last-modified` command input validation** -- Toon Claes fixed BUG() triggers and memory leaks when non-commit objects are passed to `git last-modified`.

**Submodule remote detection** -- Nasser Grainawi's fix for assuming "origin" as the default remote in submodule operations is ready with style fixes addressed.

**Sparse-checkout pattern sorting** -- amisha optimized the sparse-checkout pattern collection from O(n^2) to O(n log n) by switching to append-then-sort.

**UTF-8 diffstat display** -- LorenzoPegorari submitted a GSoC patch fixing incorrect truncation of UTF-8 filenames in diffstat output.

## On the radar

**Geometric repacking with promisor packs** -- Patrick Steinhardt's series wasn't included in 'seen' yet as Junio wants to review it further before integration.

**Hook subsystem deadlock risks** -- The discussion around stdin/stderr handling in hooks may lead to more fundamental design changes in future iterations.

**Push negotiation optimizations** -- The thread exploring inefficient data transfer during branch creation may lead to protocol improvements.