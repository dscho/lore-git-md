Here's the daily digest for April 6, 2026:

## The day in brief

A moderately busy day with 70 emails across 23 threads, featuring significant progressions in several key areas. The fsmonitor Linux implementation reached a major milestone with maintainer approval, while parallel hooks and promisor file handling moved closer to merging. Notable discussions included architectural decisions around ODB transactions and bare repository test modernization.

## Notable threads

**fsmonitor Linux implementation finalized**  
Paul Tarjan's v13 series bringing Linux fsmonitor support to parity with Windows/macOS has received maintainer approval from Junio Hamano. The implementation uses inotify with careful handling of edge cases (watch limits, unmounts, queue overflows) and includes memory leak fixes, Windows pthread emulation, and test improvements. Junio confirmed the resolution of a split-index/index.skipHash interaction issue, endorsing the test-focused solution. This marks the completion of cross-platform fsmonitor support after 13 iterations.

**Parallel hooks ready for merge**  
Adrian Ratiu's parallel hook execution series (v6) has been queued for merging by Junio after final build system adjustments from Patrick Steinhardt. The implementation provides comprehensive parallelization controls while maintaining serial execution guarantees for commit-related hooks. The series introduces multiple configuration layers including global, per-hook, and per-event job counts, with support for "-1" to use all available cores.

**Promisor file handling during repacks**  
Lorenzo Pegorari's GSoC work on preserving promisor file metadata during repacks received detailed review from Junio Hamano. While Tian Yuchen confirmed fixes for a memory leak and syntax error, Junio requested architectural changes to replace `strbuf_split_max()` with `string-list` for line parsing. The discussion clarified that modification timestamps serve as fetch-time proxies for debugging. The series now awaits a v4 addressing these review points.

**Bare repository test modernization**  
Johannes Schindelin's effort to prepare tests for potential `safe.bareRepository=explicit` default sparked discussion about implementation strategy. Junio proposed setting `safe.bareRepository=all` globally in test-lib.sh under `WITH_BREAKING_CHANGES` rather than modifying individual tests. This would minimize test churn while still supporting the eventual behavior change, paralleling how tests handle default branch names.

**ODB transaction handling in git diff**  
A thread about NULL ODB handling in `git diff --no-index` reached consensus on Justin Tobler's approach to avoid transactions entirely for hashing operations. Junio Hamano rejected Jeff King's alternative of generalized NULL handling, agreeing that hashing paths shouldn't use ODB transactions when there's no object database to write to. Luca Stefani verified Tobler's "hash-only" variant works as expected.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**Cache-tree validation fix** -- David Lin addresses an inverted condition in `cache_tree_fully_valid()` that caused incorrect validation when tree objects were missing, with Derrick Stolee confirming the regression originated in commit 062b914c84.

**Promisor-remote config fix** -- Trieu Huynh corrects submodule context handling when reading `promisor.quiet` config during lazy fetches, ensuring submodule settings override superproject values.

**Windows version requirements updated** -- Johannes Schindelin and Matthias Aßhauer's series raising Git for Windows' minimum version to 8.1 was accepted by Junio, removing legacy code paths for older Windows versions.

**parse-options subcommand typo fixes** -- A patch improves error messages when users mistype subcommands with single dashes, with Junio requesting test additions and commit message standardization.

**fast-export output ordering** -- Raymond E. Pasco identified a case where path deletions that are prefixes of added paths could be emitted in wrong order, with Elijah Newren confirming fast-import's last-entry-wins semantics make ordering significant.

## On the radar

**git stash argument parsing** -- Mirko Faina noted `--include-untracked` is valid for the "show" subcommand, potentially affecting Deveshi Dwivedi's patch to auto-assume "push" when unambiguous flags are present. Maintainers will need to weigh in on this edge case.