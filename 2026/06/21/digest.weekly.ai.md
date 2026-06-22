# The Git Project Mailing List -- Weekly Digest (2026/06/15 -- 2026/06/21)

## The period in brief

This week (2026/06/15--2026/06/21) saw **moderate to heavy traffic** (595 emails across 139 threads on 6 active days), with a **feature freeze for Git 2.55** declared mid-week. The tone was **productive and collaborative**, with several long-running efforts reaching key milestones: `git history squash` landed, ODB abstraction work advanced, and multiple security-hardened features cleared final review. The standout developments were **Taylor Blau’s `--delta-islands`/`--path-walk` integration**, **Harald Nordgren’s `git branch --delete-merged` series**, and **Patrick Steinhardt’s ref backend consolidation**. A quiet but notable trend: **GSoC projects (remote object metadata, cascading indentation) are maturing rapidly**, with both now technically complete and awaiting final polish.

---

## Key developments

### `git history squash` lands in `next`

Harald Nordgren’s four-patch series implementing `git history squash <range>`—a tool to fold a commit range into its oldest commit while preserving descendant history—**landed in `next`** after 16 iterations and a design pivot from `git rebase --squash`. The command avoids the repeated conflict stops of a rebase-based approach by computing a single three-way merge of the entire range against upstream. Key features include:
- **Ref safety**: Rejects operations with refs pointing inside the squashed range by default, with an `advice.historyUpdateRefs` hint to use `--update-refs=head`.
- **Authorship preservation**: Maintains original authorship for all commits in the range.
- **Message editing**: Supports `--reedit-message` to seed the editor with all folded-in messages (mirroring `git rebase -i` behavior).
The series touches `builtin/history.c`, `advice.c`/`advice.h`, and adds 340 lines of test coverage in `t/t3455-history-squash.sh`. All prior objections (efficiency, UX, edge cases) are resolved, and the patches are now queued for `master`.

---

### ODB abstraction reaches mechanical completion

Patrick Steinhardt’s **17-part v3 series** converting the packed object storage backend into a proper `struct odb_source` reached **mechanical completion**, with all callbacks wired up and MIDX interfaces refactored. The series represents the **final large-scale ODB restructuring** Patrick plans to undertake, but its progress is blocked by **unresolved refdb lifecycle issues** (memory leaks, duplicate refdb creation) in a separate 11-patch fix series. Junio Hamano and Justin Tobler provided surface-level reviews confirming the type-system alignment, but the series remains in limbo until the refdb fixes land. The work is critical for **pluggable object storage backends** and reftable integration.

---

### `git refs` subcommands merged

Patrick Steinhardt’s five-patch series adding `create`, `delete`, `update`, and `rename` subcommands to `git refs` **landed in `next`**, consolidating functionality previously scattered across `git-update-ref` and `git-symbolic-ref`. The design debate about **atomicity vs. backward compatibility** was resolved in favor of retaining `update`’s implicit creation behavior (for script compatibility) while adding a dedicated `create` subcommand for explicit atomic creation. The series improves discoverability and reduces fragmentation, aligning with Patrick’s broader ref backend modernization efforts. No behavioral regressions were introduced, and the implementation is now stable.

---

### `--delta-islands`/`--path-walk` integration completes

Taylor Blau’s four-patch v3 series enabling **sequential compatibility** between `--delta-islands` and `--path-walk` in `git pack-objects` **landed in `next`** after empirical validation. The implementation allows bitmaps to accelerate object counting (with path-walk as a one-way fallback) while respecting delta-islands during path-walk traversal via tree-depth recording. Performance tests in `p5311-repack.sh` confirm compression benefits persist when bitmaps are enabled (e.g., 558.4M → 164.4M in the fluentui repository). The series touches `pack-objects.c` and adds the new `p5311` perf test, with no on-disk format changes. All prior review feedback (Derrick Stolee, Junio) is incorporated.

---

### `git branch --delete-merged` nears completion

Harald Nordgren’s **16-iteration series** adding `git branch --delete-merged` (safe automated branch cleanup) is now **technically complete**, with all prior feedback addressed:
- **Stacked-branch safety**: Branches used as upstreams for unmerged stacked branches will **abort deletion by default** with a clear error message (e.g., "Cannot delete branch 'feature1' because 'feature2' depends on it").
- **Per-branch opt-out**: `branch.<name>.deleteMerged` config to exclude specific branches.
- **`--dry-run`**: Preview functionality to simulate deletions.
The series is ready for final review, with no known blocking issues. Phillip Wood’s high-weight feedback has been incorporated, and the implementation matches project API consistency expectations.

---

### Remote object metadata queries reach final polish

Pablo Sabater’s **12-patch v13 series** implementing `git cat-file --batch-command` for querying object metadata from remotes is now **technically complete**, with all substantive concerns resolved except one memory management nit. The series introduces:
- **Dynamic capability-based validation**: Prevents information leaks by validating format placeholders against advertised capabilities.
- **Strict protocol v2 enforcement**: Only "size" is advertised initially; "objecttype" returns empty strings.
- **Multi-object batching**: 10,000 objects/request to reduce network overhead.
All core maintainers (Junio, Patrick Steinhardt, Jonathan Tan, Christian Couder, Taylor Blau) have approved the approach, and the patches are rebased on current `master`. The series is security-hardened, with input size limits (8K URL length) and buffer overflow protection throughout.

---

### Cascading indentation for `git log --graph` advances

Pablo Sabater’s v6 series implementing **cascading indentation for visual roots** in `git log --graph` resolved its last architectural blocker by introducing a clean abstraction layer in `revision.c` that insulates `graph.c` from walker internals. The series addresses Jeff King’s planned removal of `revs->commits` and adds comprehensive test coverage for edge cases (filtered parents, boundary commits). Junio’s latest review requests two follow-up improvements:
1. A defensive `BUG()` check in `revision_has_more_commits()`.
2. Replacing direct `prio_queue` access in `revision_has_commits_after()` with `prio_queue_for_each()`.
The visualization logic (patch 3/3) remains uncontested, and the series is now technically complete pending these hardening changes.

---

### Worktree diff performance regression fixed

A performance investigation into `git diff` slowdowns in secondary worktrees (223.3ms vs 3.4ms in the main worktree) culminated in Jeff King’s **minimal patch** to remove the `#ifdef USE_NSEC` guard in `read-cache.c:is_racy_stat()`. The change enables nanosecond-aware timestamp comparisons unconditionally, eliminating racy entries at the source by leveraging existing nanosecond precision infrastructure. Testing confirms the problem "goes away" with this approach, which Junio Hamano acknowledged as "cute" (mild approval). The patch is small (8 lines removed, 1 line added) and directly addresses the root cause—racy-git scenarios forcing full content checks—without architectural changes or performance tradeoffs.

---

## In brief

**`git cat-file --batch-command` remote object info** -- Pablo Sabater’s security-hardened series for querying object metadata from remotes **landed in `next`** after 13 iterations. The implementation replaces static allow-lists with dynamic capability-based validation and includes fixes for memory leaks in the transport helper. -- *Security, GSoC*

**Windows large-object support** -- Johannes Schindelin’s 8-patch series converting `unsigned long` to `size_t` for object sizes on Windows (where the two types differ) **landed in `next`**. The series addresses 4GB+ object support by systematically updating object-file, ODB, and hash algorithm code paths. -- *Portability, Windows*

**SHA-256 interoperability** -- Brian M. Carlson announced that SHA-1/SHA-256 interoperability now passes its test suite, with core functionality (shallow clones, submodules, partial clones) working. However, **packfile URIs and promisor remotes remain unsupported** due to architectural constraints. -- *Cryptography, Interoperability*

**`Assisted-by:` trailer proposal** -- Marius Spix’s RFC proposing an `Assisted-by:` trailer for AI-assisted commits **sparked heated debate**. Kristoffer Haugsbakk argued that Git has historically rejected project-specific trailers, while others noted the proposal would force Git to make policy decisions (e.g., auto-propagating "taint" during merges). No consensus yet. -- *Attribution, AI*

**MIDX incremental repack regression** -- Taylor Blau’s 3-patch series fixed a regression in `git multi-pack-index write --incremental --base`, restoring reachability closure for bitmaps. The fix ensures the selected base is respected, with new test coverage in `t5319-multi-pack-index.sh`. -- *Performance, Packfiles*

**`git replay --linearize`** -- Toon Claes’s series to flatten merge commits into linear history **landed in `next`**. The feature provides a server-side alternative to `git rebase --no-rebase-merges` and includes root commit handling and test coverage. -- *History Editing*

**`git history reword` leak fixed** -- Kaartic Sivaraam’s patch plugged a memory leak in `git history reword` when handling encoded commit messages. The fix adds a missing `repo_unuse_commit_buffer` call. -- *Memory Safety*

**`git rebase --squash` architectural debate** -- Harald Nordgren’s proposal to add `--squash` to `git rebase` faced efficiency concerns (repeated conflict resolution) and UX gaps (no interactive message editing). Patrick Steinhardt endorsed `git history squash` as a more flexible alternative. -- *History Editing*

**`git update-ref --rename` deprecated** -- Junio’s standalone `--rename` proposal for `git update-ref` was **deprecated in favor of `git refs rename`**, reflecting a broader push to consolidate ref-related commands. -- *Ref Management*

**`fetch.followRemoteHEAD` global config** -- Matt Hunter’s seven-patch series adding a global fallback for `remote.<name>.followRemoteHEAD` **landed in `next`**. The implementation warns for invalid values and aligns with existing config patterns. -- *Configuration*

**Ref backend refactoring** -- Patrick Steinhardt’s 10-part v4 series modernizing Git’s reference backend infrastructure **addressed Jeff King’s architectural concerns** about recursive initialization. The series introduces a targeted `ignore_refs` flag to selectively disable "onbranch" config includes during ref store setup. -- *Ref Management*

**`ignore_case` libification** -- Justin Tobler and Tian Yuchen’s two-patch series moving the global `ignore_case` variable into `struct repo_config_values` **landed in `next`**. The series eliminates a global and ties the configuration value to the repository instance it was read from. -- *Code Cleanup*

**`SubmittingPatches` design review expectations** -- Junio’s patch formalizing design justification requirements for patch resubmissions **landed in `master`**. The update instructs contributors to defend design choices on the list before revising implementations. -- *Documentation*

**`git repo info` path formatting** -- A GSoC participant’s three-patch v7 series introducing standardized path formatting for repository information **landed in `next`**. The series adds `append_formatted_path()` to consolidate duplicated logic in `rev-parse` and exposes the repository’s Git directory and common directory paths in both absolute and relative formats. -- *Scriptability*

**Shell completion dotfile hiding** -- Zakariyah Ali’s v3 series aligning Git’s path completion with shell conventions (hiding dotfiles unless explicitly requested) **remains blocked** on Junio’s request for a cleaner two-patch progression. -- *UX, Completion*

**Commit-reach optimization** -- Tian Yuchen’s six-patch series optimizing `paint_down_to_common()` for one-sided histories **landed in `next`**, introducing an early-exit condition that terminates the merge-base walk when one side’s queue is exhausted. Benchmarks show 100–1000× speedups for large one-sided histories. -- *Performance*

**Config-include-disabling feature retracted** -- Derrick Stolee formally retracted his three-patch series proposing mechanisms to disable Git config include directives (`GIT_CONFIG_INCLUDES=0` and `--no-includes`). The retraction followed substantive concerns from Jeff King about security trade-offs and workflow breakage. -- *Configuration*

**`git branch --prune-merged`** -- Harald Nordgren’s `hn/branch-prune-merged` topic (adding `--prune-merged` to `git branch`) is **controversial due to UX debates** but mature. The series is ready for final review. -- *Branch Management*

**`git log --follow` for non-linear history** -- Martin von Zweigbergk’s `mv/log-follow-mergy` topic (extending `git log --follow` to handle non-linear history) **remains stalled** and needs attention. -- *History Traversal*

**Configurable lock timeout** -- Jonathan Tan’s `jt/config-lock-timeout` topic (adding a configurable timeout for config file locking) **remains stalled**. -- *Concurrency*

---

## Looking ahead

### Topics likely to dominate next week

1. **Git 2.55 stabilization**: With the feature freeze declared, expect a **flurry of regression fixes** and documentation updates. Junio’s "What’s cooking" report highlighted several topics in `next` that need final polish (e.g., `git history squash`, `git refs` subcommands).
2. **ODB abstraction follow-ups**: Patrick Steinhardt’s **11-patch refdb fix series** is the blocker for the ODB abstraction work. The fixes address memory leaks and duplicate refdb creation, and their resolution will unblock a critical architectural effort.
3. **Pinned references design**: Erik Östlund’s proposal for "pinned references" (a ref + expected OID invariant) remains in early-stage design discussion. The next steps will likely involve a **strawman implementation** or a pivot to a new command/option.
4. **macOS CI hang fix**: Michael Montalbo’s proposed fix for macOS CI hangs in `t5551` and `t5559` (packing refs and reducing parallelism) needs a maintainer decision. Jeff King’s concern about masking the underlying HTTP/2 deadlock may require deeper investigation.
5. **`git log --graph` cascading indentation**: Pablo Sabater’s v6 series needs **final hardening changes** (defensive `BUG()` check, `prio_queue_for_each()`) before the abstraction layer can be considered fully reviewed. The visualization logic is uncontested, so expect rapid progress once these are addressed.

### Ongoing efforts with no recent progress

- **`mm/diff-process-hunks`**: Michał Kępień’s topic allowing external tools to act as hunk providers **remains stalled** due to security and performance concerns. This is a potentially powerful but complex feature.
- **`git replay --linearize`**: Thomas Koutcher’s `git replay --linearize` subcommand (to linearize a commit history) **needs broader review**. The command is experimental and lacks user feedback.
- **`git history drop`**: Patrick Steinhardt’s new `git history drop` subcommand (to remove a commit and replay its descendants) **needs feedback**. The command is part of an ongoing effort to provide more flexible history editing tools.