Here's the Git mailing list daily digest for May 19, 2026:

## The day in brief

A busy Tuesday with 149 emails across 30 threads, featuring significant progress on multiple fronts. Key highlights include Christian Couder's promisor remote configuration series reaching completion, Taylor Blau's incremental MIDX repacking optimization series being approved, and Patrick Steinhardt's `the_repository` removal work in the setup subsystem concluding. Several performance optimizations and documentation improvements also moved forward.

## Notable threads

**Promisor remote configuration finalized** -- Christian Couder's URL-based auto-configuration series for promisor remotes (v3) has completed all technical review cycles and is ready for merging. The series introduces `promisor.acceptFromServerUrl` config for pattern-based remote trust, with strict security constraints around URL matching and automatic remote name generation. The implementation includes comprehensive test coverage and documentation updates, building on the previously approved LOP (Large Object Promisors) series to provide a complete solution for promisor remote management.

**Incremental MIDX repacking approved** -- Taylor Blau's 16-patch series enabling incremental MIDX repacking has received final approval from Jeff King and is now queued for integration. The changes allow Git to maintain MIDX chains through both append-only growth and geometric compaction strategies, significantly improving scalability for large repositories. Performance benchmarks show 4x-6x speedups in test repositories with millions of objects. This completes a multi-part effort Taylor has been working on to improve Git's pack management infrastructure.

**`the_repository` removal in setup subsystem** -- Patrick Steinhardt's 18-patch series converting the setup subsystem to use explicit repository parameters instead of the global `the_repository` variable has concluded with Junio's approval. The changes systematically eliminate one of the largest remaining uses of global state in Git's codebase while maintaining identical behavior. The series follows a proven three-step conversion methodology and includes thread-safety improvements by removing static buffers in path handling functions.

**Negotiation controls for fetch/push** -- Derrick Stolee's series introducing `--negotiation-include` and `--negotiation-restrict` options has reached its final form (v6) after thorough review. These controls help optimize object transfer in monorepos by allowing users to specify which refs should always be included or restricted during negotiation. The implementation now properly handles config overrides and push integration, with all technical feedback addressed. Matthew Cheetham provided the final review confirmation needed before maintainer consideration.

**Commit-graph edge case fix** -- Jeff King identified and fixed a rare but problematic interaction between commit-graph lazy-loading and `git clone --dissociate`. The patch adds a fallback path that reads commit objects directly from disk when graph-based lookups fail, preventing checkout failures in edge cases. Rasmus Villemoes confirmed the fix works in real-world use, and Junio approved the implementation after discussing hash algorithm handling and code organization improvements.

## In brief

**`git describe` pattern matching fix** -- Jacob Keller's patch to make `git describe --contains --all` properly reject `--match` and `--exclude` patterns rather than silently ignoring them gained real-world validation from Tuomas Ahola, who independently encountered the same issue.

**`git log --graph` visualization** -- Pablo Sabater's series to improve root commit display in `--graph` output has settled on using cascading indentation after Junio expressed concerns about ambiguity with fixed indentation approaches. The implementation preserves the critical invariant that vertical alignment always indicates true ancestry.

**`git-jump` auto mode improvements** -- Greg Hurrell's patch to add automatic mode selection to `git-jump` gained consensus on restructuring as an explicit `auto` mode rather than implicit behavior. Jeff King approved the core functionality while remaining neutral on potentially adding staged whitespace checking.

**Documentation style clarification** -- Jean-Noël Avila's man page standardization effort sparked discussion about whether `[synopsis]` formatting should be limited to SYNOPSIS sections or can extend to in-body command examples. The thread revealed a need to clarify project documentation guidelines.

**Performance optimizations** -- Multiple optimization threads progressed: René Scharfe's strbuf_grow overflow checking refactoring was approved; Kristofer Karlsson's commit-reach.c improvements received technical review; and Taylor Blau posted a new series optimizing pack-bitmap-write performance with measured 4x-6x speedups.

**External notes command support** -- Siddh Raman Pant posted a 9-part series introducing the ability to fetch notes from long-running external processes, with timeout handling and comprehensive test coverage. The feature addresses TOCTOU races in distributed environments while maintaining security through protected config options.

## On the radar

**`git stash -p` fsmonitor optimization** -- A new patch aims to speed up `git stash -p` with fsmonitor by avoiding full refreshes via optimized index creation. Initial benchmarks show dramatic improvement from 34.774s to 0.659s in large repos.

**`--track=fetch` edge cases** -- Harald Nordgren's series adding fetch-on-checkout behavior needs to address two final edge cases raised by Junio: overlapping remote-tracking refs and missing HEAD handling. The series is otherwise technically complete.

**Priority queue optimizations** -- Discussion continues between Kristofer Karlsson and Jeff King about extending priority queue usage in revision walking, with both approaches showing 3-5x speedups but differing in architectural tradeoffs.