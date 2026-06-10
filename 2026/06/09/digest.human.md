# Here's the Git mailing list digest for June 9, 2026:

**The day in brief.** A busy Wednesday with 109 emails across 31 threads, featuring significant progress on several major efforts including Patrick Steinhardt's ODB abstraction work, performance optimizations for `git describe` and the finalization of the `git cat-file --batch-command` security hardening. Junio's "What's cooking" report provides a comprehensive overview of the current development landscape.

## Notable threads

### ODB abstraction reaches milestone

Patrick Steinhardt's 17-part series converting packed object storage to use `struct odb_source` reached v2, representing the final major restructuring in the multi-year effort to enable pluggable storage backends. The series received architectural approval from Karthik Nayak, with only minor formatting nits addressed in this iteration. During implementation, Patrick uncovered deeper refdb architectural issues that will require a follow-up 11-patch series to resolve memory leaks and initialization problems.

### Security-hardened cat-file ready for merge

Pablo Sabater's security-focused `git cat-file --batch-command` series completed its 12th iteration with all technical concerns resolved. The final version implements dynamic capability-based validation of format placeholders, replacing static allow-lists. Reviewers (Junio, Chandra Pratap) focused on last-minute type safety and documentation nits, indicating the series is now merge-ready after extensive refinement of this security-sensitive feature.

### Promisor remote configuration finalized

Christian Couder's promisor remote auto-configuration series reached consensus in its v4 iteration after resolving the final discussion point about configuration behavior. Toon Claes accepted Christian's rationale for maintaining fall-through behavior between `acceptFromServer` and `acceptFromServerUrl` configurations, allowing layered security policies. The series now provides comprehensive URL pattern matching with strict security constraints while preserving workflow flexibility.

### Worktree metadata tracking evolves

Norbert Kiesel's worktree metadata series expanded in v2 to include both timestamp tracking and description management. Phillip Wood's review suggested reorganizing the series into three logical parts (descriptions, creation timestamps, sorting) with clearer commit messages explaining each change's motivation. The implementation maintains backward compatibility while adding creation/last-updated timestamps and user-provided descriptions with sorting capabilities.

## In brief

**Ref-filter performance fix** -- Tamir Duberstein's patch restores prefix-scoped iteration in ref-filter, fixing a regression that caused ~25x slowdowns when listing refs with many loose refs.

**Describe optimization** -- Tamir Duberstein's v2 patch speeds up `git describe` by 17x (172ms->10ms) by limiting ref iteration to tags when `--all` isn't specified.

**Reachability refactoring** -- A patch consolidates reachability functions in commit-reach.c, removing redundant `get_reachable_subset()` in favor of the newer `tips_reachable_from_bases()`.

**Update-ref rename option** -- Junio proposes adding `--rename` to `git update-ref` for low-level ref renaming with reflog preservation, deferring batch mode support.

**Trailer URL parsing fix** -- Jeff King proposes modifying `find_separator()` to exclude lines containing `://` from trailer parsing, fixing URL misinterpretation.

## On the radar

**Branch deletion regression** -- Harald Nordgren's `git branch --delete-forked` series fixed a critical regression in v14 but remains blocked on platform-dependent test failures affecting both CI and local environments.

**History reword behavior** -- Pablo Sabater's RFC for `git history reword` continues evolving, now considering Junio's suggestion for a cross-command `--avoid-unnecessary-rewrite` option rather than command-specific behavior changes.

**Config includes disablement** -- Derrick Stolee's series to disable config includes is reconsidering scope after Jeff King identified workflow breakage risks, potentially shifting toward a comprehensive sandboxing solution.