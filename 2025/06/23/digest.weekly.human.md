# Git Development Digest - 2025/06/23 -- 2025/06/29

**The week in brief.** A moderately busy week with 342 emails across 94 threads saw steady progress on multiple fronts alongside several significant feature completions. The standout developments were Taylor Blau's MIDX/cruft pack optimizations reaching merge readiness, the finalization of stash import/export functionality, and resolution of long-standing issues with `git apply --intent-to-add`. Performance optimizations dominated the week's technical discussions, while platform-specific challenges (particularly on signal handling and Windows filesystems) required ongoing attention.

## Key developments

### MIDX and cruft pack optimizations finalized

Taylor Blau's performance optimization series for MIDX and cruft pack interaction during repacking reached its final form this week. The v6 iteration addressed an edge case with incremental MIDX writing where cruft packs could incorrectly appear in the MIDX under specific conditions. The fix properly traverses all MIDX layers (including base_midx chains) when collecting pack names, ensuring correct exclusion of cruft packs where possible. The series introduces a new `repack.midxMustContainCruft` config (default true) that allows excluding cruft packs from MIDX when safe, reducing bloat while maintaining reachability guarantees. Production testing shows 5-20% speed improvements, with all patches receiving thorough review from Junio Hamano and Elijah Newren.

### Stash import/export functionality merges

After 8 iterations, Phillip Wood's series implementing stash transfer between repositories via `git stash export` and `import` commands was approved for merging. The implementation preserves stash message structure and commit topology using `refs/stash-export/`, with the final version including clearer documentation, fixed memory ownership in `write_commit_with_parents()`, and additional argument validation. The 369 lines of test coverage and reusable infrastructure like the GET_OID_GENTLY flag demonstrate the careful engineering behind this long-requested feature that makes stashes portable across repositories.

### `git apply --intent-to-add` fixed after years

Johannes Altmanninger addressed long-standing issues with the effectively non-functional `git apply --intent-to-add` (`-N`) option. The five-patch series starts with safety checks (erroring out when used outside a repository), fixes the core issue by properly reading the index in intent-to-add mode, ensures only new files get intent-to-add markers, adds comprehensive tests, and clarifies documentation. This well-scoped fix resolves a real usability issue that's lingered since Git 2.19, with each patch addressing a specific aspect of the problem through focused changes and thorough test coverage.

### Git daemon signal handling debate

An extensive discussion emerged around git-daemon's signal handling, with three approaches now under consideration to address zombie process cleanup on OpenBSD and crashes on AIX. Carlo Marcelo Arenas Belón's SA_RESTART toggling approach competes with Phillip Wood's self-pipe alternative and a minimal EINTR handling solution. The thread revealed architectural tradeoffs - SA_RESTART requires platform-specific handling while self-pipe would restructure the event loop but eliminate those variations. Windows-specific signal handling quirks further complicate the decision, with Johannes Schindelin's input likely needed before final resolution.

### Bloom filter optimization for pathspecs

Lidong Yan's series enabling bloom filter acceleration for multiple pathspec queries in revision traversal reached its final polishing phase. The implementation introduces `struct bloom_keyvec` to manage multiple bloom keys while maintaining the single-pathspec fast path. Junio Hamano confirmed the existing tests already cover correctness by comparing bloom-filtered results with non-bloom results, while requesting additional guardrails against false negatives. Benchmark results show the optimized version now matches single-pathspec performance, addressing a notable gap in large repository workflows.

## In brief

**Promisor-remote protocol refactoring** -- Christian Couder's v5 series enhancing the promisor-remote protocol with configurable field validation was approved, allowing servers to advertise additional remote attributes via `promisor.sendFields` while enabling client-side validation through `promisor.checkFields`.

**IMAP send fixes and features** -- Aditya Garg's 19-iteration series combining critical fixes (configuration parser regression, memory leaks) with major new functionality (OAuth2.0 support, PLAIN authentication) was approved for merging.

**`git fetch --prune` optimization** -- A performance optimization reduced dangling ref detection from O(M*N) to O(N*logM) by sorting the prune list to enable binary search, cutting runtime from 410 seconds to under 1 second in test cases.

**Windows filesystem regressions** -- Investigations continued into lock file/ref update errors in newer Git versions and "Permission denied" issues with DFS-mounted network drives, highlighting persistent Windows platform challenges.

**String-list test modernization** -- Shejialuo's 8-part series converted shell-based tests to C unit tests while improving the core string-list implementation, fixing sign comparison warnings and removing historical cruft.

**Git-log documentation standardization** -- Jean-Noël Avila's 9-patch series converting git-log documentation to standardized AsciiDoc format was approved, affecting Git's second-largest man page and related files.

**Parse-options type safety** -- A 6-part series refactored Git's parse-options infrastructure to introduce type precision handling for integer-based options, adding bounds checking and overflow protection.

**Interactive diff context configuration** -- Phillip Wood's v3 series implementing diff context configuration for interactive patch commands (`add -p`, `checkout -p`, etc.) appears ready for merging after addressing all review feedback.

## Looking ahead

**SHA-256 by default** -- brian m. carlson's foundational series continues cooking in Junio's integration branches, preparing for Git 3.0's breaking changes, with a v2 expected soon.

**Refs list command proposal** -- The `git refs list` discussion continues off-thread, with documentation improvements being made to the existing `git-for-each-ref` as part of the review process.

**Reftable compaction** -- Patrick Steinhardt's reftable work was recently mentioned in "What's cooking" as needing review attention in the coming weeks.