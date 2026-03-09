# Git Development Digest - 2025/06/23 -- 2025/06/29

**The week in brief.** A moderately busy week with 342 emails across 94 threads saw steady progress on multiple fronts alongside several long-running efforts reaching completion. The standout developments were Taylor Blau's MIDX/cruft pack optimizations merging to 'next', the finalized stash import/export functionality, and Christian Couder's promisor-remote protocol enhancements. Performance optimizations dominated the week's technical discussions, with significant work on bloom filters, fetch-prune speedups, and for-each-ref regressions. Windows-specific issues and signal handling debates also featured prominently.

## Key developments

### MIDX and cruft pack optimizations land

Taylor Blau's performance optimization series for MIDX and cruft pack interaction during repacking reached its final form and was merged to 'next'. The series addresses edge cases with incremental MIDX writing where cruft packs could incorrectly appear in the MIDX under specific conditions. The implementation introduces a new `repack.midxMustContainCruft` config (default true) that allows excluding cruft packs from MIDX when safe, reducing bloat while maintaining reachability guarantees. Production testing shows 5-20% speed improvements. The v6 iteration addressed the last remaining style and correctness issues identified by Jeff King and Junio Hamano, particularly around proper traversal of MIDX chains via `m->base_midx`.

### Stash import/export functionality finalized

Phillip Wood's v8 patches implementing stash transfer between repositories were approved for merging after addressing all feedback. The series introduces `git stash export` and `import` commands that preserve stash message structure and commit topology via `refs/stash-export/`. The final version included clearer documentation, fixed memory ownership in `write_commit_with_parents()`, and additional argument validation. With 369 lines of test coverage and reusable infrastructure like the GET_OID_GENTLY flag, this marks the successful conclusion of a long-running effort to make stashes portable across repositories. The implementation follows Git's established patterns for data transfer operations while handling the unique topology of stash commits.

### Promisor-remote protocol enhancements complete

Christian Couder's 5-part series enhancing the promisor-remote protocol reached its v5 iteration and was approved. The changes allow servers to advertise additional remote attributes (like partialCloneFilter and token) via `promisor.sendFields` while enabling client-side validation through `promisor.checkFields`. Significant internal refactoring replaced strvec usage with explicit struct promisor_info, providing a foundation for future protocol extensions. This work is part of Git's Large Object Promisors (LOP) effort and saw thorough review from Patrick Steinhardt, Karthik Nayak, and Justin Tobler. Junio Hamano raised final documentation questions about field ordering and mandatory-to-understand markers before merging.

### Bloom filter optimization for pathspecs

Lidong Yan's series optimizing bloom filters for multiple pathspec queries in revision traversal saw significant progress, addressing performance gaps when using commands like `git log -- file1 file2`. The implementation introduces `struct bloom_keyvec` to manage multiple bloom keys while maintaining the single-pathspec fast path. Junio Hamano confirmed the existing tests already cover correctness by comparing bloom-filtered results with non-bloom results, while requesting additional guardrails against false negatives. Benchmark results show the optimized version now matches single-pathspec performance, with the series in final polishing addressing minor points about memory management patterns.

### Git daemon signal handling debate

An extensive discussion emerged around git-daemon's signal handling, with three approaches now under consideration: Carlo Marcelo Arenas Belón's SA_RESTART toggling, a self-pipe alternative, and Phillip Wood's minimal EINTR handling solution. The debate revealed architectural tradeoffs - SA_RESTART requires platform-specific handling while self-pipe would restructure the event loop but eliminate those variations. The thread represents an important design decision point for reliable signal handling in the daemon, particularly around zombie process cleanup on OpenBSD and crashes on AIX. Windows-specific signal handling quirks were also discussed, with Carlo suggesting to involve Johannes Schindelin for additional platform expertise.

## In brief

**`git fetch --prune` optimization** -- O(M*N) to O(N*logM) speedup for dangling ref detection reduces runtime from 410 seconds to under 1 second in test cases while maintaining identical functionality.

**Windows filesystem regressions** -- Multiple reports identified lock file and ref update issues in Git 2.46+ on Windows, with Brian m. carlson pointing to related Git for Windows issue #3825 for DFS-mounted network drive problems.

**`for-each-ref` performance regression** -- Andrzej Dębski reported significant slowdowns with ~65K tags traced to reachability checking introduced in commit cbfe360b140, impacting large-scale repository operations.

**Interactive diff context configuration** -- Phillip Wood's v3 series implementing configurable diff context for interactive patch commands (`add -p`, `checkout -p`, etc.) appears ready for merging with comprehensive test coverage.

**`git apply --intent-to-add` fix** -- Johannes Altmanninger's series addresses long-standing issues where `-N` created an empty index except for new files, making it useless outside blank repositories since Git 2.19.

**String-list test modernization** -- Shejialuo's 8-part series converts shell-based tests to C unit tests while improving the core string-list implementation, fixing sign comparison warnings and removing historical cruft.

**Git-log documentation standardization** -- Jean-Noël Avila's 9-patch conversion of git-log documentation to standardized AsciiDoc format was approved, affecting Git's second-largest man page.

**Bash completion security issue** -- Ondrej Pohorelsky reported command injection via branch names containing `>` during tab completion, with consensus emerging to fix in `git-completion.bash`.

## Looking ahead

**SHA-256 by default** -- brian m. carlson's foundational series continues cooking in Junio's integration branches, preparing for Git 3.0's breaking changes, with a v2 expected soon.

**Refs list command proposal** -- The `git refs list` discussion continues off-thread, with documentation improvements being made to the existing `git-for-each-ref` as part of the review process.

**Comment character auto-selection** -- Phillip Wood surfaces deeper design questions about how `core.commentChar="auto"` should interact with `core.commentstring` configuration across multiple commands.