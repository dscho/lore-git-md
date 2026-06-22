# The Git Project Mailing List -- Weekly Digest (2026/06/15 -- 2026/06/21)

## The period in brief

This was a **highly productive week** on the Git mailing list, with **496 emails across 139 threads** covering seven active days. Traffic was **heavy and substantive**, featuring the **completion of several long-running series**, **major architectural advances**, and **significant design debates** about reference management, history editing, and object storage. The most important developments: **`git history squash` landed in `next`**, **`git branch --delete-merged` reached its 16th iteration with all design issues resolved**, **ODB abstraction work advanced toward pluggable backends**, and **Git v2.55.0-rc1 was released**, marking the start of the stabilization period for the next major release. Readers who followed the daily digests closely will find little new here; those who skipped them should focus on the **history editing tools**, **reference management consolidation**, and **ODB abstraction progress**—these are the efforts that will shape Git’s evolution in the coming months.

---

## Key developments

### `git history` subcommands mature: `squash` and `drop` reach key milestones

The experimental `git history` command, introduced in Git 2.53 as a vehicle for more flexible history editing, saw **two major subcommands reach critical milestones** this week. Harald Nordgren’s **`git history squash`**—a tool to fold a commit range into its oldest commit while preserving descendant history—**landed in `next`** after 13 iterations and a decisive pivot from a proposed `git rebase --squash` option. The implementation avoids the efficiency pitfalls of rebase-based approaches (repeated conflict resolution) by computing a single three-way merge of the entire range against upstream, and it handles edge cases like fixup-commit sequences and merge commits correctly. The design also resolved a key safety concern: by default, the command **rejects operations where refs point into the squashed range**, with an `advice()` message directing users to `--update-refs=head` if they want to retarget those refs to the squashed commit.

Separately, Patrick Steinhardt’s **`git history drop`**—a subcommand to remove a commit while replaying its descendants—**completed its technical implementation** and was introduced in Junio’s “What’s cooking” report. The series modernized Git’s reset machinery as part of the effort and includes **537 lines of test coverage** for safety mechanisms. Both subcommands are part of a broader effort to provide more flexible and safer alternatives to `git rebase -i` and `git filter-branch`, and they reflect the project’s increasing comfort with experimental commands that can evolve based on user feedback.

---

### `git branch --delete-merged` reaches technical completion

Harald Nordgren’s **16-iteration series** adding `git branch --delete-merged`—a tool for safe automated cleanup of merged local branches—**reached technical completion** this week, with all design and implementation feedback addressed. The command deletes branches that have been merged into a specified commit (defaulting to `HEAD`), with extensive safety mechanisms: it **aborts if the branch is checked out**, **warns if the branch has been pushed to a remote with divergent history**, and **allows per-branch opt-out via config** (`branch.<name>.deleteMerged`). The series also includes a **`--dry-run` preview** and a **`--force` override** for the safety checks.

The final design hurdle—**stacked-branch safety**—was resolved with a pragmatic approach: the command **aborts by default** if the branch being deleted is used as an upstream for another unmerged branch (e.g., `feature2` stacks on `feature1`), with a clear error message. This avoids the complexity of automatic retargeting (e.g., recursion in chains like `b1→b2→b3`) while preserving backward compatibility for future enhancements. The series is now **ready for final review** and likely to graduate to `next` in the coming week.

---

### ODB abstraction work advances toward pluggable backends

Patrick Steinhardt’s **ongoing effort to abstract Git’s object database (ODB)** saw **significant progress** this week, with the **mechanical completion of a 17-patch series** converting the packed object storage backend into a proper `struct odb_source`. The series, now in its third iteration, systematically refactors the ODB interface to enable pluggable backends (e.g., cloud storage, alternative formats) and is a prerequisite for Patrick’s broader ref backend modernization efforts, including the reftable backend.

The series is **mechanically complete**—all callbacks are wired up, and the MIDX interface has been refactored to use the concrete packed source type—but it remains **blocked by unresolved refdb architectural issues** (memory leaks, duplicate refdb creation, initialization ordering) that Patrick is addressing in a separate 11-patch fix series. Junio Hamano acknowledged the series as **ready for integration once the refdb issues are resolved**, and the work represents the **final large-scale ODB restructuring** Patrick plans to undertake. The mechanical changes are uncontroversial, but the architectural dependencies highlight the complexity of modernizing Git’s storage layer while maintaining backward compatibility.

---

### Reference management consolidation: `git refs` subcommands land

Patrick Steinhardt’s **five-patch series** adding reference-writing subcommands (`create`, `delete`, `update`, `rename`) to `git refs` **landed in `next`** this week, consolidating functionality previously scattered across `git-update-ref` and `git-symbolic-ref`. The series improves discoverability and reduces fragmentation in Git’s reference management interface, particularly for scripting and automation.

The design debate about **atomicity vs. backward compatibility** was resolved in favor of retaining `update`’s implicit creation behavior (for script compatibility) while adding a dedicated `create` subcommand for explicit atomic creation. This compromise reflects the project’s preference for **gradual evolution** over breaking changes, even when the latter might offer cleaner semantics. The series also aligns with Patrick’s broader ref backend modernization efforts, including the ongoing ODB abstraction work, and it sets the stage for future improvements like **atomic multi-ref transactions**.

---

### Git v2.55.0-rc1 released, marking the start of the stabilization period

Junio C Hamano **released Git v2.55.0-rc1** on June 17, incorporating **460 non-merge commits from 82 contributors**. The release includes **new features** like parallel hook execution, the `git format-rev` builtin, and fsmonitor support for Linux, along with **significant performance optimizations** and **internal refactoring** (e.g., ODB abstraction, `the_repository` removal, Rust integration). The sheer volume of changes—including ongoing efforts like the ODB abstraction and history editing tools—makes this a **significant release**, and the focus now shifts to **regression fixes and documentation updates**.

The -rc1 tarballs are available for testing, and the stabilization period will continue until the final release, expected in late July. The release notes highlight the **experimental status** of `git history` and the **performance improvements** in `git log --graph` and `git branch --remotes`, among other changes.

---

### Security-hardened remote object metadata queries reach final polish

Pablo Sabater’s **12-patch series** implementing `git cat-file --batch-command` for querying object metadata from remotes **reached its 13th iteration** this week, with all substantive concerns resolved except one memory management nit. The series introduces **dynamic capability-based format placeholder validation**, **strict protocol v2 enforcement**, and **multi-object batching** (10,000 objects/request) to reduce network overhead. It is **security-hardened**, with input size limits (8K URL length) and buffer overflow protection throughout, and it enables querying object sizes without full downloads—a boon for large-repository workflows.

All core maintainers (Junio, Patrick Steinhardt, Jonathan Tan, Christian Couder, Taylor Blau) have approved the approach, and the series is **rebased on current `master`**. The final nit—a `static` declaration in `parse_cmd_remote_object_info()`—is the last hurdle before the series can graduate to `next`.

---

### `git log --graph` cascading indentation clears architectural hurdle

Pablo Sabater’s **six-iteration series** implementing cascading indentation for visual roots in `git log --graph` **resolved its last architectural blocker** this week by introducing a clean abstraction layer in `revision.c` that insulates the graph renderer from the walker’s internal data structures. The series addresses a long-standing limitation in Git’s graph visualization: commits with excluded parents (e.g., `git log --graph --exclude=origin/main`) were previously rendered with incorrect indentation, making the graph harder to interpret.

The abstraction layer—**conditionally approved by Junio**—requires a follow-up patch for defensive `BUG()` checks, but the core visualization logic (patch 3/3) is now protected and uncontested. The series includes **comprehensive test coverage** for edge cases like filtered parents and boundary commits, and it is **ready for final review** once the hardening changes are addressed.

---

## In brief

**`git pack-objects` gains full `--delta-islands`/`--path-walk` compatibility** -- Taylor Blau’s four-patch v3 series resolved the long-standing limitation where `--path-walk` could not leverage bitmaps or delta-islands. The implementation enables sequential compatibility: bitmaps accelerate object counting (with path-walk as a one-way fallback), while delta-islands are respected during path-walk traversal via tree-depth recording. Empirical validation confirms that path-walk’s compression benefits persist when bitmaps are enabled, and the series is now **merge-ready**.

**Windows large-object support inches closer** -- Johannes Schindelin’s **8-patch series** converting `unsigned long` to `size_t` for object sizes on Windows (where the two types differ) received **Junio’s ack** for its delta-handling patch. The series, now **rebased and conflict-free**, addresses 4GB+ object support by systematically updating object-file, ODB, and hash algorithm code paths. All production changes have been validated in Git for Windows, and the series is **poised for `next`** pending final review.

**SHA-256 interoperability hits a milestone** -- Brian M. Carlson announced that **SHA-1/SHA-256 interoperability** now passes its test suite, with core functionality (shallow clones, submodules, partial clones) working. However, **critical limitations** remain: packfile URIs and promisor remotes are unsupported due to architectural constraints (e.g., unmapped objects in disconnected packs). The 200+ patch series is **not targeted for Git 3.0**, but the milestone marks progress toward a long-term goal.

**`Assisted-by:` trailer proposal divides contributors** -- A **controversial RFC** from Marius Spix proposed adopting an `Assisted-by:` trailer for AI-assisted commits, mirroring the Linux kernel’s recent policy. Kristoffer Haugsbakk **pushed back hard**, arguing that Git has historically rejected project-specific trailers (citing a 2013 `Fixes:` rejection) and that the proposal would force Git to make policy decisions (e.g., auto-propagating "taint" during merges). The debate touches on **Git’s role in attribution standards** and whether it should normalize `--trailer` behavior to match `--signoff`.

**MIDX incremental repack regression fixed** -- Taylor Blau’s **3-patch series** fixed a regression in `git multi-pack-index write --incremental --base`, restoring reachability closure for bitmaps. The regression (introduced in Git 2.55-rc1) caused the command to ignore custom bases, breaking incremental repacking. The fix ensures the selected base is respected, and the series is **ready for review**.

**Commit-reach optimization series lands with 500× speedup** -- Tian Yuchen’s six-patch series optimizing `paint_down_to_common()` for one-sided histories **landed** this week, introducing an early-exit condition that terminates the merge-base walk when one side’s queue is exhausted. The optimization targets a known bottleneck in repositories with asymmetric histories (e.g., import grafts or shallow histories), with benchmarks showing **100–1000× speedups** for large one-sided histories (2.6M commits). The series is **well-structured** and includes test coverage from Elijah Newren’s parallel work.

**Worktree diff performance investigation yields minimal root-cause fix** -- A performance investigation into `git diff` slowdowns in secondary worktrees (223.3ms vs 3.4ms in the main worktree) culminated in Jeff King’s **minimal patch** to remove the `#ifdef USE_NSEC` guard in `read-cache.c:is_racy_stat()`. The change enables nanosecond-aware timestamp comparisons unconditionally, eliminating racy entries at the source by leveraging existing nanosecond precision infrastructure. The patch is small (8 lines removed, 1 line added) and directly addresses the root cause—racy-git scenarios forcing full content checks—without architectural changes or performance tradeoffs.

**`git branch --delete-merged` stacked-branch safety finalized** -- Harald Nordgren’s v16 series implementing safe automated local branch cleanup now uses a **simplified stacked-branch protection**: branches used as upstreams for unmerged stacked branches will **abort deletion by default** with a clear error message. This avoids the complexity of retargeting (e.g., recursion in chains like `b1→b2→b3`) while preserving backward compatibility for future enhancements.

**Shell completion dotfile hiding blocked on structural overhaul** -- Zakariyah Ali’s v3 series aligning Git’s path completion with shell conventions (hiding dotfiles unless explicitly requested) remains **blocked** on Junio’s request for a cleaner two-patch progression: (1) refactor the AWK script in `__git_index_files` for readability without changing behavior, and (2) implement dotfile hiding on top of that refactored base.

**`git repo info` path formatting series ready for pickup** -- A GSoC participant’s three-patch v7 series introducing standardized path formatting for repository information is now **ready for Junio’s pickup**. The series adds `append_formatted_path()` to consolidate duplicated logic in `rev-parse` and exposes the repository’s Git directory and common directory paths in both absolute and relative formats for scriptability.

**Ref backend refactoring advances toward final review** -- Patrick Steinhardt’s 10-part v4 series modernizing Git’s reference backend infrastructure has addressed Jeff King’s architectural concerns about recursive initialization during ref store setup. The series introduces a targeted `ignore_refs` flag in `struct config_options` to selectively disable "onbranch" config includes during ref store initialization, preventing recursive ref store access while preserving other include conditions.

**Documentation update on patch reroll cadence ready for merging** -- Weijie Yuan’s single-patch v3 series updating `SubmittingPatches` to codify implicit norms around patch reroll cadence is now **uncontested and ready for Junio to pick up**. The patch advises contributors to batch rerolls (no more than one per day by default) and signal which parts of a series will become obsolete when substantial rework is needed.

**Build system quietness patches merged-ready for gitk and git-gui** -- Harald Nordgren’s two-patch series aligning gitk and git-gui’s translation catalog generation with core Git’s quiet build conventions is now **procedurally ready for merging**. The patches remove `--statistics` from `msgfmt` invocations and conditionally suppress "Generating catalog" messages when `make -s` is used.

**`ignore_case` libification merged** -- Justin Tobler and Tian Yuchen’s two-patch series moving the global `ignore_case` variable into `struct repo_config_values` was **accepted by Junio**. The series introduces a `repo_ignore_case()` getter and migrates all usage sites, eliminating a global and tying the configuration value to the repository instance it was read from.

---

## Looking ahead

The next week is likely to see **continued stabilization work** for Git 2.55, with a focus on **regression fixes** and **documentation updates**. Key topics to watch:

- **`git history` subcommands**: Both `squash` and `drop` are in `next` and may graduate to `master` if no regressions are reported. User feedback on these experimental commands will shape their future evolution.
- **ODB abstraction**: Patrick Steinhardt’s refdb fix series is the last blocker for the ODB abstraction work, and its resolution will unlock progress toward pluggable backends.
- **Reference management**: The `git refs` subcommands are in `next` and may see follow-up work to improve atomicity or add multi-ref transactions.
- **SHA-256 interoperability**: While not targeted for Git 3.0, the recent milestone may generate renewed discussion about the long-term roadmap for hash algorithm support.
- **Pinned references**: Erik Östlund’s proposal for "pinned references" (a ref + expected OID invariant) remains in early-stage design discussion, and the next week may see a concrete proposal or patches.