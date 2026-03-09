# Git Mailing List Digest - 2025/04/28 -- 2025/05/04

## The week in brief

A busy week with 326 emails across 105 threads saw significant progress on several fronts. Key developments included the completion of Patrick Steinhardt's object store API cleanup, finalization of `git gc` decomposition into granular maintenance tasks, and resolution of Outlook Message-ID handling in `git send-email`. The week also featured important discussions around deprecation policies, rebase reflog corruption fixes, and path-walk delta compression optimizations.

## Key developments

### Object store API cleanup finalized

Patrick Steinhardt's multi-part effort to clean up Git's object store API reached completion this week. The series systematically replaced `repo_has_object_file()` with a more explicit `has_object()` API, providing better control over promisor object fetching and packfile reloading. After thorough review from Karthik Nayak and Junio Hamano, the final version introduced a `HAS_OBJECT_FETCH_PROMISOR` flag to preserve promisor object handling where needed. This marks a significant milestone in the ongoing `the_repository` removal effort, with all patches now queued for merging.

### `git gc` decomposition completed

After years of incremental work, Patrick Steinhardt's series to decompose `git gc` into standalone maintenance tasks was finalized. The implementation adds worktree pruning and rerere garbage collection as configurable tasks, with auto-execution thresholds (`maintenance.worktree-prune.auto` and `maintenance.rerere-gc.auto`) matching existing GC behavior. Derrick Stolee approved the technical approach after the series simplified the rerere GC counting phase in response to Junio Hamano's performance concerns. This completes the transition from monolithic GC to granular, independently configurable maintenance operations.

### Outlook Message-ID resolution

Aditya Garg's work to handle Outlook's Message-ID rewriting behavior in `git send-email` concluded with the implementation of a tri-state configuration (`always|never|auto`). The solution automatically detects Outlook SMTP hosts while allowing manual override for enterprise configurations, addressing a long-standing pain point where Outlook servers would rewrite Message-IDs and break email threads. Junio Hamano accepted the design despite preferring a simpler boolean interface, recognizing the need for flexibility in real-world deployments.

### Path-walk delta compression approved

Derrick Stolee's path-walk delta compression series received final approval for inclusion in 'next'. The feature introduces path-based delta compression through `--path-walk` and `pack.usePathWalk` config, with optimizations for shallow clones. Performance data shows dramatic improvements (100x size reduction in some cases) for repositories with path-based similarity patterns. Taylor Blau completed a thorough review of documentation, code structure, and test strategy, clearing the way for integration pending final verification.

### Deprecation infrastructure established

Junio Hamano proposed and finalized a standardized deprecation mechanism, demonstrated by applying it to `git whatchanged`. The series generalizes warning infrastructure from `git pack-redundant`, systematically removes `whatchanged` references, and introduces a `--i-still-use-this` opt-in requirement. This establishes a template for future command deprecations while taking a cautious approach - first gathering usage data via forced opt-in before full removal. The implementation includes conditional compilation via `WITH_BREAKING_CHANGES` and documents the planned removal in BreakingChanges.adoc.

## In brief

**Rebase reflog corruption** -- Investigation confirmed a heap-use-after-free in `git rebase --rebase-merges` when processing complex merge sequences, with Phillip Wood proposing conversion to `strbuf` for safer reflog message handling.

**Index-pack delta chain detection** -- Derrick Stolee's bugfix series addressing incorrect cycle detection in REF_DELTA chains was accepted, including a new `test-tool pack-deltas` helper for targeted test cases.

**BSD errno handling** -- Consensus formed on using `lstat()` rather than platform-specific errno checks for symlink operations in `packed-backend.c`, improving portability.

**`git apply --intent-to-add`** -- Analysis revealed index corruption cases when using `-N`, with Raymond Pasco identifying an additional edge case where partial diffs are incorrectly applied.

**Windows `git svn` maintenance** -- Johannes Schindelin acknowledged precarious maintenance state for Windows support, suggesting WSL as workaround and linking to issue about potential deprecation.

**Documentation standardization** -- Junio Hamano continued Jean-Noël Avila's synopsis-style conversion effort, updating multiple man pages to consistent formatting and backquoting.

**Scalar maintenance control** -- Series added `--no-maintenance` flags to `scalar register` and `clone`, though Junio Hamano raised architectural concerns about implementation location.

**Assertion handling** -- Discussion expanded to Windows-specific considerations, with Johannes Schindelin defending assert-suppression for MSVC CI builds and Patrick Steinhardt proposing systematic conversion of `assert()` calls.

**Looking ahead**

**Rebase reflog fix implementation** -- With consensus reached on using `strbuf` for safer reflog message storage, expect Phillip Wood's implementation patch soon.

**Rustification debate** -- Randall Becker's concerns about platform support remain unresolved as Ezekiel Newren's Rust integration work continues.

**ODB abstraction** -- Patrick Steinhardt's object storage layer refactoring may require performance validation following Junio Hamano's feedback on rerere GC impacts.