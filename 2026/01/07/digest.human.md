Here's the Git mailing list daily digest for January 7, 2026:

---

### The day in brief
A busy day with 86 emails across 27 threads, dominated by major feature work on submodule path encoding and ODB abstraction. The submodule gitdir path encoding series reached its final form with v8, while Patrick Steinhardt's ODB refactoring continued to progress through review. Notable discussions included finalizing whitespace checks for documentation and resolving a security advisory discrepancy.

---

### Notable threads

**Submodule gitdir path encoding finalized**  
Adrian Ratiu's 12-patch series implementing configurable submodule gitdir paths via `extensions.submodulePathConfig` reached its eighth and final iteration, incorporating all review feedback. The feature allows custom gitdir locations while preventing filesystem conflicts through a four-tier fallback system (plain/encoded/numbered/hashed paths) with RFC3986 encoding for case-folding protection. Junio Hamano has acked the series for merging after extensive review from Patrick Steinhardt and others. The implementation includes comprehensive test coverage and an atomic migration command (`git submodule--helper migrate-gitdir-configs`) to convert existing repositories.

**ODB abstraction advances**

Patrick Steinhardt's object database refactoring work saw significant progress with multiple series moving forward. His 7-patch v4 series improving object info handling received final review approval from Karthik Nayak, demonstrating 9% speedups in disk usage calculations. Meanwhile, the larger packfile store relocation effort (moving MIDX tracking to odb_source) continued review with Toon Claes confirming the architectural soundness of changes. These changes are part of GitLab's ongoing effort to enable pluggable ODB backends.

**Whitespace checking for documentation**

Adrian Ratiu's new `tab-between-non-ws` whitespace rule (likely to be renamed `tab-instead-of-space`) received detailed review from Johannes Sixt and Junio Hamano. The check detects when tabs appear between non-whitespace characters in documentation where spaces were likely intended, particularly useful for AsciiDoc files. Discussion focused on test improvements and naming clarity, with the implementation tracking both position and visual columns to handle tab expansion. This complements Jean-Noël Avila's documentation standardization effort.

**git-history command infrastructure**

Patrick Steinhardt sent v8 of his `git-history` series, extracting replay functionality into a library interface. The changes move core revision walking logic from `builtin/replay.c` to a new `replay_revisions()` function with clean parameter passing. While not changing behavior, this refactoring enables the planned history` command's reword and split subcommands. The series builds on Git's replay machinery while drawing inspiration from Jujutsu's history editing capabilities.

---

### In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**git reset documentation polish** -- Final stylistic improvements to the `git reset` man page following Julia Evans' pedagogical rewrite, now ready for 'next'.

**PID tracking for lock files** -- Paulo Casaretto's bugfix adds `~pid.lock` files to help diagnose stale locks, with simplified `core.lockfilePid` configuration.

**Memory management fixes** -- Junio marked Patrick Steinhardt's two-patch series (commit-graph and submodule ODB leaks) ready for 'next' after thorough review.

**HTTP auth tests** -- Ashlesh Gawande's v3 adds .netrc test cases for 401/403 responses, now with clearer commit message explaining coverage gaps.

**add-p hunk display** -- Abraham Samuel Adekunle's GSoC work on showing previous hunk decisions explores compact notation options to preserve 80-column width.

---

### On the radar

**CVE version range clarification** -- The advisory for CVE-2025-48385 (bundle-uri) may incorrectly list versions predating the feature's introduction in v2.38.0.

**the_repository removal effort** -- Outreachy intern Bello Olamide reports on architectural challenges around config variable deglobalization during weeks 3-4.

**promisor remote enhancements** -- Christian Couder's LOP series sees review feedback on filter combination logic and server-advertised filter storage.

**git pack-refs NFS performance** -- Investigation continues into mmap bottlenecks, with Patrick Steinhardt advocating reftable backend as long-term solution over NFS optimizations.

**core.attributesFile migration** -- Phillip Wood proposes structured three-phase approach to move config settings while minimizing disruption during `the_repository` removal.

--- 

The day's activity reflects steady progress on several major technical initiatives while addressing documentation quality, performance optimizations, and long-term architectural improvements. The submodule path encoding work stands out as reaching completion after extensive iteration, while the ODB abstraction and `the_repository` removal efforts continue their multi-stage evolution.