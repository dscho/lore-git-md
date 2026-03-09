Here's the daily digest for January 7, 2026:

**The day in brief.** A busy day with 86 emails across 27 threads, dominated by major feature finalizations and architectural work. The standout developments include Adrian Ratiu's submodule gitdir path encoding series reaching completion, Patrick Steinhardt's ODB abstraction patches getting final approval, and ongoing refinements to documentation and whitespace handling. Junio Hamano remains active in review despite personal circumstances.

**Notable threads**

**Submodule gitdir path encoding finalized**  
Adrian Ratiu's long-running series implementing configurable submodule gitdir paths via `extensions.submodulePathConfig` has reached its final form with v8. The feature allows custom gitdir paths while preventing filesystem conflicts through a four-tier fallback system (plain/encoded/numbered/hashed paths) with case-folding protection. Key additions in this version include relative path test coverage, improved error messages, and renamed configuration (`init.defaultSubmodulePathConfig`). The series has been acked by Junio after extensive review from Patrick Steinhardt and others, representing a significant enhancement to submodule flexibility and robustness.

**ODB abstraction patches approved**  
Patrick Steinhardt's 7-patch series refactoring object info handling for the ODB abstraction effort received final approval from Karthik Nayak. The changes deliver measurable performance improvements (9% speedup in disk usage calculations) while cleaning up interfaces through enum-based type tracking and simplified return values. The v4 iteration addressed all review feedback, including restructuring loose object handling to use a single exit path. This work prepares the codebase for future pluggable ODB backends while maintaining current functionality.

**git-history command takes shape**  
Patrick Steinhardt's experimental `git history` command progresses with v8 of the series, now focusing on the `reword` subcommand that allows in-memory commit message editing. The implementation builds on Git's replay infrastructure rather than the sequencer, enabling bare repository operation and optional branch updates. D. Ben Knoble provided final documentation polish, though the deferred `split` subcommand remains desired functionality. The changes demonstrate careful attention to error handling and interface design for this alternative history editing approach.

**In brief**  
**PID tracking for lock files** -- Paulo Casaretto's patch adds `~pid.lock` companion files to help diagnose stale locks, now simplified to a boolean `core.lockfilePid` config after review feedback.

**Memory leak fixes** -- Patrick Steinhardt's two-patch series fixing commit-graph writing leaks and submodule ODB leaks is marked for 'next' by Junio after thorough review.

**Clar test framework updated** -- Patrick Steinhardt's 3-patch series updating Git's embedded clar test framework with improved integer handling completes its review cycle.

**git repo info --keys** -- Lucas Seiki Oshiro's feature adding key listing to `git repo info` addresses final review feedback about format handling consistency.

**Whitespace checking refined** -- Adrian Ratiu's new `tab-between-non-ws` check (likely to be renamed) for documentation files receives test improvement suggestions from Johannes Sixt.

**On the radar**  
**the_repository removal effort** -- Bello Olamide's internship work on config handling faces design questions about initialization timing, with Phillip Wood proposing a structured three-phase migration approach.

**git pack-refs NFS performance** -- Investigation continues into mmap bottlenecks, with perf data confirming memory operations dominate runtime but no clear solution emerging yet.

**git add -p UI improvements** -- Abraham Samuel Adekunle's GSoC work on hunk decision feedback explores compact notation options to preserve 80-column convention.