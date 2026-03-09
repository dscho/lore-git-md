# Git Mailing List Digest - 2025/04/28 -- 2025/05/04

## The week in brief

A busy week with 326 emails across 105 threads, featuring significant progress on several major initiatives. The standout developments include the completion of Patrick Steinhardt's object store API cleanup and `git gc` decomposition into granular maintenance tasks, resolution of Outlook Message-ID handling in `git send-email`, and confirmation of a memory corruption bug in `git rebase --rebase-merges`. The week also saw Junio Hamano establish standardized deprecation patterns with `git whatchanged` and Derrick Stolee's path-walk delta compression receive final approval.

## Key developments

### Object store API cleanup finalized

Patrick Steinhardt's multi-part series to clean up the object store API reached completion this week, systematically replacing `repo_has_object_file()` with the more explicit `has_object()` API. The final version introduced a `HAS_OBJECT_FETCH_PROMISOR` flag to preserve promisor object fetching capabilities where needed, addressing Junio Hamano's concerns about default behaviors. Karthik Nayak provided thorough review, particularly on patches 6-12 which dealt with subtle design considerations around promisor object handling. This marks a significant milestone in the ongoing `the_repository` removal effort, with the changes now queued for merging.

### `git gc` decomposed into maintenance tasks

After multiple iterations, Patrick Steinhardt's series to fully decompose `git gc` into standalone maintenance tasks was approved. The final version added worktree pruning and rerere garbage collection as configurable tasks, with `maintenance.worktree-prune.auto` and `maintenance.rerere-gc.auto` thresholds. Derrick Stolee confirmed readiness after the implementation simplified the rerere GC approach in response to Junio Hamano's performance concerns. This completes a multi-year effort to replace monolithic GC with granular, while maintaining behavioral parity with existing workflows.

### Rebase reflog memory corruption identified

A serious heap-use-after-free in `git rebase --rebase-merges` was confirmed through Kristoffer Haugsbakk and Phillip Wood's investigation. Sanitizer output pinpointed the issue to `ctx->reflog_message` pointing to freed memory during sequencer operations. The discussion reached consensus on converting to a `strbuf` for proper string ownership management, with Jeff King (Peff) providing the technical analysis. This explains intermittent reflog corruption reports and provides a clear path to fixing a subtle but impactful bug affecting merge rebases.

### Path-walk delta compression approved

Derrick Stolee's path-walk delta compression series received final approval for inclusion in 'next' after addressing all technical feedback. The feature introduces path-based compression grouping through `--path-walk` and `pack.usePathWalk` config, with optimizations for shallow clones. Performance data shows dramatic improvements (100x size reduction in some cases) for repositories with path-based similarity patterns. Taylor Blau completed a final review pass, focusing on documentation and test strategy, though requested until Tuesday to finish before final integration.

### Standardized deprecation mechanism established

Junio Hamano proposed and finalized a standardized deprecation pattern, demonstrated by applying it to `git whatchanged`. The series extracts existing warning infrastructure from `git pack-redundant` into reusable helper, then requires `--i-still-use-this` for `whatchanged`. This establishes a cautious "gauge the usage" phase via forced opt-in before potential removal, while providing comprehensive documentation updates and conditional compilation via `WITH_BREAKING_CHANGES`. The pattern is now available for future command deprecations as Git 3.0 planning progresses.

## In brief

**Outlook Message-ID handling** -- Aditya Garg's work on `git send-email` reached completion with enterprise configurability via `--smtp-outlook-id-tweak` (always|never|auto), addressing both standard and custom domain cases.

**Index-pack delta chain detection** -- Derrick Stolee's bugfix series fixing incorrect cycle detection in REF_DELTA chains was accepted, including a new `test-tool pack-deltas` helper for targeted test cases.

**Optional file handling** -- Junio Hamano resurrected his `:(optional)` prefix syntax series to address `.git-blame-ignore-revs` use cases while providing a general mechanism for optional file specifications.

**BSD errno handling** -- Consensus formed on using `lstat()` rather than platform-specific checks for symlink operations, avoiding EFTYPE/EMLINK differences across BSD variants.

**`git apply --intent-to-add`** -- Investigation revealed index corruption cases when using `-N`, with Raymond Pasco identifying an edge case where partial diffs are incorrectly applied.

**Windows `git svn` maintenance** -- Johannes Schindelin acknowledged precarious Windows support status, suggesting WSL as workaround while considering deprecation due to lack of maintainers.

**Documentation standardization** -- Junio Hamano continued Jean-Noël Avila's synopsis-style conversion effort across multiple commands, with Eric Sunshine reviewing backquote consistency.

**Reftable test modernization** -- Seyi Chamber's conversion of reftable tests to Clar framework progressed through v3, with Patrick Steinhardt providing extensive style feedback.

**Merge visualization** -- Johannes Sixt raised thoughtful design questions about generalizing Britton Kerin's 3-pane approach beyond 2-parent merges.

## Looking ahead

**Rebase reflog fix implementation** -- With consensus on the strbuf conversion approach, expect Phillip Wood's patch soon to address the memory corruption issue.

**Bitmap lookup tables** -- Taylor Blau's series to enable bitmap lookup tables by default may see movement after limited reviewer engagement despite resolving earlier concerns.

**Rustification debate** -- Randall Becker's platform support concerns remain unresolved as Ezekiel Newren's Rust integration work continues.

**ODB abstraction** -- Patrick Steinhardt's object storage layer refactoring nears completion but may require performance validation per Junio's feedback on rerere GC.