# Here's the daily digest for May 22, 2026:

**The day in brief.** Friday saw moderate traffic with 94 emails across 21 threads, featuring several significant developments. The git-gui bugfix series reached final approval, Harald Nordgren's `git branch --prune-merged` landed in v11 with all requested safety checks, and Derrick Stolee's `--path-walk`/`--filter` integration series concluded technical discussions. Meanwhile, debates continued around external process timeouts and structural diff proposals.

## Notable threads

### git-gui bugfix series finalized

The 12-patch series addressing repository and worktree detection edge cases in git-gui received its final approvals from maintainer Johannes Sixt. After resolving the last timing issue with `_prefix` initialization and simplifying environment variable handling, the series is ready for merging. The changes fix long-standing issues dating back to 2014/2019 regressions, particularly improving behavior in bare repositories and with detached HEAD` states.

### `git branch --prune-merged` reaches v11

Harald Nordgren's safe branch cleanup feature completed its 11-iteration journey with comprehensive safety mechanisms now in place. The final version adds `--dry-run` support and push-tracking comparisons to avoid deleting trunk branches, while maintaining all prior protections (checked-out branches, missing upstreams). Junio Hamano clarified earlier review comments didn't require restarting from v9, clearing the way for potential merging after extensive collaborative refinement.

### `--path-walk`/`--filter` integration ready

Derrick Stolee and Taylor Blau's performance-focused series enabling full interoperability between `--path-walk` and all `--filter` options concluded technical discussions in its fifth iteration. The co-authors reached consensus on the path marking approach using leading '/' characters, with only minor test script adjustments remaining. Performance data shows dramatic speedups (72.53s->29.00s for git/git with `sparse:oid`) while maintaining correctness across all filter types and combinations.

### External process timeout debate continues

The proposed external notes helper mechanism sparked extended discussion about timeout handling, with Jeff King (Peff) and Johannes Sixt arguing against implementing timeouts in core Git. Siddh Raman Pant ultimately agreed to remove the controversial timeout functionality from v2, though the broader architectural questions about failure recovery in external processes remain unresolved. Junio Hamano suggested potential protocol-based negotiation as a middle ground.

### Structural diff RFC faces skepticism

Michael Montalbo's proposal for `diff.<driver>.process` allowing external tools to inject diff hunks received cautious feedback from Junio Hamano. While technically sound, the maintainer questioned whether the 17% blame slowdown and added complexity were justified given existing `textconv` capabilities. The author clarified the primary use case is AST/structural diffs and proposed OID-based optimizations to reduce overhead, but the feature's future remains uncertain without stronger performance improvements or use case documentation.

## In brief

**Windows signal handling clarified** -- Siddh Raman Pant cited Microsoft documentation confirming Windows lacks Unix-style signal semantics, supporting a patch to treat `SIGTERM` and `SIGKILL` identically via `TerminateProcess()`.

**`push.default=simple` docs updated** -- A documentation patch clarifying triangular workflow behavior received maintainer suggestions for broader restructuring to better explain the safety constraints.

**`receive-pack` bugfix landed** -- A 2-line change fixed interaction between `updateInstead` and `core.worktree` configurations caused by environment variable leakage from a8cc594333.

**Container zombie processes reported** -- SURA documented `git fetch` leaving zombie processes when run as PID 1 in Docker, with D. Ben Knoble referencing prior discussions about similar issues.

**Race condition in rapid operations** -- Fabrice SALVAIR reported repository corruption during rapid `git add`/`git commit` sequences in Git 2.54, with evidence pointing to background maintenance interference.

## On the radar

**ODB abstraction progresses** -- Patrick Steinhardt's repository initialization refactoring series completed with all technical and editorial requirements met, centralizing object database creation as part of the broader storage backend effort.

**External notes v2 pending** -- Siddh Raman Pant's external notes command support removed timeout functionality per community feedback and will return as v2 with revised focus on protocol robustness.