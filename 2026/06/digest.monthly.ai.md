# Git Mailing List Monthly Digest -- 2026 June

## The period in brief

June 2026 was an **exceptionally productive month** for the Git project, with **2,184 emails across 591 threads** spanning 24 active days. Traffic was **heavy and technically substantive**, featuring the **completion of several multi-year architectural efforts**, **major performance optimizations**, **security hardening**, and **significant usability improvements**. The most important developments: **the ODB abstraction work reached a major milestone**, **`git history` subcommands (`squash`, `drop`) landed in `next`**, **`git branch --delete-merged` completed its 18-iteration review cycle**, **Git v2.55.0-rc1 was released**, and **foundational ref backend refactoring** modernized Git’s reference storage infrastructure. Readers who followed the daily digests closely will find little new here; those who skipped them should focus on the **ODB abstraction progress**, **history editing tools**, **reference management consolidation**, and **performance optimizations**—these efforts will shape Git’s evolution in the coming year.

---

## Key developments

### ODB abstraction reaches major milestone: pluggable backends enabled

Patrick Steinhardt’s **multi-year effort to abstract Git’s object database (ODB)** reached a **critical inflection point** in June, with the **mechanical completion of the packed object storage conversion** to `struct odb_source`. The 17-patch series, now in its third iteration, systematically refactors the ODB interface to enable **pluggable backends** (e.g., cloud storage, alternative formats) and is a prerequisite for Patrick’s broader ref backend modernization efforts, including the reftable backend.

The series is **mechanically complete**—all callbacks are wired up, and the MIDX interface has been refactored to use the concrete packed source type—but remains **blocked by unresolved refdb architectural issues** (memory leaks, duplicate refdb creation, initialization ordering) that Patrick is addressing in a separate 11-patch fix series. Junio Hamano acknowledged the series as **ready for integration once the refdb issues are resolved**, and the work represents the **final large-scale ODB restructuring** Patrick plans to undertake. The mechanical changes are uncontroversial, but the architectural dependencies highlight the complexity of modernizing Git’s storage layer while maintaining backward compatibility.

The ODB abstraction work is **foundational for Git’s future scalability**, enabling alternative storage backends (e.g., cloud-native object stores) and paving the way for more flexible ref storage (e.g., reftable). The June progress marks the **end of the mechanical refactoring phase** and the start of **architectural stabilization**, with the refdb fix series expected to land in July.

---

### `git history` subcommands mature: `squash` and `drop` land in `next`

The experimental `git history` command, introduced in Git 2.53 as a vehicle for more flexible history editing, saw **two major subcommands reach critical milestones** in June. Harald Nordgren’s **`git history squash`**—a tool to fold a commit range into its oldest commit while preserving descendant history—**landed in `next`** after 13 iterations and a decisive pivot from a proposed `git rebase --squash` option. The implementation avoids the efficiency pitfalls of rebase-based approaches (repeated conflict resolution) by computing a single three-way merge of the entire range against upstream, and it handles edge cases like fixup-commit sequences and merge commits correctly. The design also resolved a key safety concern: by default, the command **rejects operations where refs point into the squashed range**, with an `advice()` message directing users to `--update-refs=head` if they want to retarget those refs to the squashed commit.

Separately, Patrick Steinhardt’s **`git history drop`**—a subcommand to remove a commit while replaying its descendants—**completed its technical implementation** and was introduced in Junio’s “What’s cooking” report. The series modernized Git’s reset machinery as part of the effort and includes **537 lines of test coverage** for safety mechanisms. Both subcommands are part of a broader effort to provide more flexible and safer alternatives to `git rebase -i` and `git filter-branch`, and they reflect the project’s increasing comfort with experimental commands that can evolve based on user feedback.

The `git history` subcommands are **not yet stable**, but their inclusion in `next` signals the project’s intent to **gradually stabilize them** based on user feedback. The June progress represents a **major step forward** for Git’s history editing capabilities, offering more intuitive and safer alternatives to existing tools.

---

### `git branch --delete-merged` reaches technical completion

Harald Nordgren’s **18-iteration series** adding `git branch --delete-merged`—a tool for safe automated cleanup of merged local branches—**reached technical completion** in June, with all design and implementation feedback addressed. The command deletes branches that have been merged into a specified commit (defaulting to `HEAD`), with extensive safety mechanisms: it **aborts if the branch is checked out**, **warns if the branch has been pushed to a remote with divergent history**, and **allows per-branch opt-out via config** (`branch.<name>.deleteMerged=false`). The series also includes a **`--dry-run` preview** and a **`--force` override** for the safety checks.

The final design hurdle—**stacked-branch safety**—was resolved with a pragmatic approach: the command **aborts by default** if the branch being deleted is used as an upstream for another unmerged branch (e.g., `feature2` stacks on `feature1`), with a clear error message. This avoids the complexity of automatic retargeting (e.g., recursion in chains like `b1→b2→b3`) while preserving backward compatibility for future enhancements. The series is now **ready for final review** and likely to graduate to `next` in July.

The `git branch --delete-merged` feature is a **significant quality-of-life improvement** for branch management, offering a safer and more configurable alternative to manual branch cleanup. Its completion marks the end of a **long and thorough review process**, reflecting the project’s commitment to **safety and usability**.

---

### Ref backend lazy-loading refactor lands, modernizing reference storage

Patrick Steinhardt’s **11-patch series** modernizing Git’s reference backend infrastructure was **merged into `next`** in June, resolving recursive initialization issues caused by `includeif.onbranch` conditions. The redesign defers write-config parsing until the first write operation, eliminating early config reads that could trigger recursion. The series touches 34 files, including core ref backends (`files-backend.c`, `reftable-backend.c`), and introduces a recursion guard in `get_main_ref_store()`. This is a **foundational improvement** for ref backend modularity, with no user-visible behavior changes unless `includeif.onbranch` is used. The series is poised to graduate to `master` in the next integration cycle.

The ref backend refactoring is part of a **broader effort to modernize Git’s reference storage**, including the ongoing ODB abstraction work and the reftable backend. The June progress represents a **critical step forward** for Git’s reference management infrastructure, enabling more flexible and robust storage backends.

---

### Git v2.55.0-rc1 released, marking the start of the stabilization period

Junio C Hamano **released Git v2.55.0-rc1** on June 17, incorporating **460 non-merge commits from 82 contributors**. The release includes **new features** like parallel hook execution, the `git format-rev` builtin, and fsmonitor support for Linux, along with **significant performance optimizations** and **internal refactoring** (e.g., ODB abstraction, `the_repository` removal, Rust integration). The sheer volume of changes—including ongoing efforts like the ODB abstraction and history editing tools—makes this a **significant release**, and the focus now shifts to **regression fixes and documentation updates**.

The -rc1 tarballs are available for testing, and the stabilization period will continue until the final release, expected in late July. The release notes highlight the **experimental status** of `git history` and the **performance improvements** in `git log --graph` and `git branch --remotes`, among other changes.

The June release cycle reflects the **project’s ongoing commitment to innovation**, with a focus on **scalability, usability, and performance**. The stabilization period will ensure that the new features and optimizations are **robust and well-documented** before the final release.

---

### Security hardening: `git cat-file --batch-command` and reftable hardening

June saw **significant progress** in Git’s security hardening efforts, with two major series reaching critical milestones:

1. **`git cat-file --batch-command`**: Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries (e.g., object sizes) is now **technically complete and ready for merging**. The series introduces a new `remote-object-info` command that lets clients request metadata for up to 10,000 objects in a single command, reducing network overhead for partial clone workflows. The series is **security-hardened**, with strict protocol v2 enforcement, input validation, and **comprehensive test coverage** (680 lines in `t1017-cat-file-remote-object-info.sh`). The only unresolved question is **error-handling philosophy**: should the client fail explicitly when metadata is missing, or continue silently (matching local `git cat-file` behavior)? The series is expected to land in `next` in July.

2. **Reftable hardening**: Patrick Steinhardt’s **11-patch series** hardening the reftable backend against corrupted files is now **fully reviewed and ready for `next`**. The series fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in the reftable parser, and adds a **libFuzzer-based fuzzer** to prevent regressions. The fixes are **internal-only** (no user-facing changes) and critical for the reftable backend’s stability. The series is expected to graduate to `master` shortly.

These security hardening efforts reflect the **project’s ongoing commitment to robustness and safety**, particularly in areas like remote object queries and reference storage.

---

### Performance optimizations: `paint_down_to_common()`, `git log --graph`, and more

June saw **dramatic performance improvements** across multiple Git commands, driven by targeted optimizations and architectural refactoring:

1. **`paint_down_to_common()` optimization**: Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged**, delivering **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts). The series terminates early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories. The optimization is a **major performance win** for `git merge-base` in repositories with asymmetric histories.

2. **`git log --graph` cascading indentation**: Pablo Sabater’s **six-iteration series** implementing cascading indentation for visual roots in `git log --graph` **resolved its last architectural blocker** by introducing a clean abstraction layer in `revision.c`. The series addresses a long-standing limitation in Git’s graph visualization: commits with excluded parents (e.g., `git log --graph --exclude=origin/main`) were previously rendered with incorrect indentation, making the graph harder to interpret. The series is **ready for final review** and expected to land in `next` in July.

3. **`git pack-objects` `--delta-islands`/`--path-walk` compatibility**: Taylor Blau’s **four-patch v3 series** resolved the long-standing limitation where `--path-walk` could not leverage bitmaps or delta-islands. The implementation enables sequential compatibility: bitmaps accelerate object counting (with path-walk as a one-way fallback), while delta-islands are respected during path-walk traversal via tree-depth recording. The series is **merge-ready** and expected to land in `next` in July.

These performance optimizations reflect the **project’s ongoing focus on scalability and efficiency**, particularly in large repositories and complex workflows.

---

### Post-merge regressions: `git replay --linearize` and `git repack`

June also saw **two significant post-merge regressions** that required urgent attention:

1. **`git replay --linearize`**: Johannes Schindelin reported a **post-merge regression** in `git replay --linearize` (v5), where the command **silently drops commits** when replaying a single branch containing merge commits. In a real-world example (`master~2..master`), only the tip commit is replayed, and the “Git 2.55-rc2” commit is lost. The regression stems from the removal of the `replayed_base` logic in `pick_regular_commit()`. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled. This is a **critical regression** that needs urgent attention before the next release.

2. **`git repack --geometric --cruft`**: Taylor Blau’s **10-patch RFC** to combine `--geometric` and `--cruft` repack modes hit a **correctness snag**: Junio identified a flaw in the two-phase traversal logic of `--stdin-packs=follow-reachable`, where **unreachable tags and objects** may be incorrectly retained. The current implementation marks *all* tag objects in included packs as reachable, even if they’re not referenced by any ref. The series remains in **RFC limbo** until this is resolved.

These regressions highlight the **challenges of maintaining backward compatibility** while introducing new features and optimizations. The project’s **rigorous review and testing processes** help catch these issues early, but they also underscore the **importance of post-merge validation**.

---

## In brief

**ODB abstraction: `struct object_info` refactoring** -- Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use a `source` field (replacing the coarse `whence` enum) received **Junio’s conceptual approval**. The series enables **multi-source object resolution** and is a **foundational step** for pluggable ODB backends.

**`git history squash` v6** -- Harald Nordgren’s series to fold commit ranges into a single commit is **functionally complete**, with stricter input validation, support for multiple revision arguments, and the `--reedit-message` flag.

**`git log -L` range-scoped diff stat** -- Karthik Nayak’s 7-patch series extending `git log -L` to support `--stat`, `--check`, and `-G` pickaxe operations is **complete and uncontroversial**.

**`excludes_file` libification** -- Tian Yuchen’s patch moving the global `excludes_file` into `struct repo_config_values` is **ready for `next`** as part of the **libification effort** to eliminate global state.

**`git history --reword` file leak fix** -- Junio’s **bugfix patch** addresses a file stream leak in `git history --reword` that could cause problems on Windows.

**`--track=fetch` workflow debate** -- Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on **philosophical concerns** about network operations during checkout.

**CI hangs in `t5551`/`t5559`** -- Jeff King’s 3-patch series increases Apache’s `Timeout` directive to 600 seconds and isolates the expensive “many-tags” test case.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass and is **ready for merging**.

**`greplint.pl` exposes test-suite fragility** -- Junio demands an audit of all `# lint-ok:` comments to ensure they do not hide latent test bugs.

**Windows large-object support** -- Johannes Schindelin’s **8-patch series** converting `unsigned long` to `size_t` for object sizes on Windows (where the two types differ) received **Junio’s ack** for its delta-handling patch.

**SHA-256 interoperability** -- Brian M. Carlson announced that **SHA-1/SHA-256 interoperability** now passes its test suite, with core functionality (shallow clones, submodules, partial clones) working.

**`Assisted-by:` trailer proposal** -- A **controversial RFC** from Marius Spix proposed adopting an `Assisted-by:` trailer for AI-assisted commits, mirroring the Linux kernel’s recent policy.

**MIDX incremental repack regression fixed** -- Taylor Blau’s **3-patch series** fixed a regression in `git multi-pack-index write --incremental --base`, restoring reachability closure for bitmaps.

**Commit-reach optimization series lands with 500× speedup** -- Tian Yuchen’s six-patch series optimizing `paint_down_to_common()` for one-sided histories **landed**, introducing an early-exit condition that terminates the merge-base walk when one side’s queue is exhausted.

**Worktree diff performance investigation** -- Jeff King’s **minimal patch** to remove the `#ifdef USE_NSEC` guard in `read-cache.c:is_racy_stat()` enables nanosecond-aware timestamp comparisons unconditionally, eliminating racy entries at the source.

**`git branch --delete-merged` stacked-branch safety** -- Harald Nordgren’s v18 series uses a **simplified stacked-branch protection**: branches used as upstreams for unmerged stacked branches will **abort deletion by default** with a clear error message.

**Shell completion dotfile hiding** -- Zakariyah Ali’s v3 series aligning Git’s path completion with shell conventions (hiding dotfiles unless explicitly requested) remains **blocked** on Junio’s request for a cleaner two-patch progression.

**`git repo info` path formatting series** -- A GSoC participant’s three-patch v7 series introducing standardized path formatting for repository information is **ready for Junio’s pickup**.

**Ref backend refactoring** -- Patrick Steinhardt’s 10-part v4 series modernizing Git’s reference backend infrastructure has addressed Jeff King’s architectural concerns about recursive initialization during ref store setup.

**Documentation update on patch reroll cadence** -- Weijie Yuan’s single-patch v3 series updating `SubmittingPatches` to codify implicit norms around patch reroll cadence is **uncontested and ready for Junio to pick up**.

**Build system quietness patches** -- Harald Nordgren’s two-patch series aligning gitk and git-gui’s translation catalog generation with core Git’s quiet build conventions is **procedurally ready for merging**.

**`ignore_case` libification** -- Justin Tobler and Tian Yuchen’s two-patch series moving the global `ignore_case` variable into `struct repo_config_values` was **accepted by Junio**.

**`git ls-files` optimization** -- Tamir Duberstein’s patch filtering pathspecs before `lstat` operations shows **50x speedup** (60.7s → 1.06s) in large repositories.

**Subcommand autocorrection** -- Jiamu Sun’s typo correction for Git subcommands like `git remote` was merged to `seen` pending final style fixes.

**Test infrastructure hardening** -- Patrick Steinhardt’s series enforces valid TAP output in tests and standardizes CI environments across GitLab and GitHub.

**Worktree metadata tracking** -- Norbert Kiesel proposed adding creation timestamps to worktrees, sparking debate about utility versus Junio Hamano’s preference for simplicity.

**Repository initialization** -- Patrick Steinhardt’s 8-patch series centralizing object database setup during repo initialization completed review.

**`diff.<driver>.process` tests** -- Michael Montalbo committed to transitioning from Python to C-based tests after Windows CI issues were identified.

**`git describe` optimization** -- A patch limiting ref iteration to tags by default shows **3.1x speedup** (196ms → 63ms) in repos with many non-tag refs.

**Documentation standards** -- Junio Hamano updated `SubmittingPatches` with better typo-fix guidance and cover letter advice.

**Config key validation** -- Harald Nordgren’s `git config` syntax hint series concluded after refining API design to separate parsing and validation.

**Stash performance** -- Adam Johnson’s `git stash -p` optimization replacing `read-tree` with `unpack_trees()` shows **34.774s→0.659s improvements**.

**Promisor remote configuration** -- Christian Couder’s URL-based auto-configuration patches implement secure URL pattern matching through `promisor.acceptFromServerUrl` config and are now in `next`.

**Trailer documentation** -- Kristoffer Haugsbawk’s 11-patch series comprehensively improved `git-interpret-trailers` documentation, standardizing terminology and clarifying format restrictions.

**Worktree metadata** -- Norbert Kiesel’s series added creation timestamps and free-form descriptions to worktrees with sorting options.

**Reftable refactoring** -- A patch consolidated reachability functions in `commit-reach.c`, removing redundant `get_reachable_subset()` in favor of `tips_reachable_from_bases()`.

**Update-ref rename option** -- Junio proposed adding `--rename` to `git update-ref` for low-level ref renaming with reflog preservation.

**Merge-ort hardening** -- Elijah Newren’s series hardened merge-ort against corrupt trees and improved directory/file conflict detection with binary search in `verify_cache()`.

**External diff driver RFC** -- Michael Montalbo’s RFC introducing `diff.<driver>.process` configuration now has 708 lines of test coverage and addresses all substantive review feedback.

**Documentation synopsis conversion** -- Jean-Noël Avila’s octothorpe handling solution completed this piece of the standardization effort.

---

## Looking ahead

The next month is likely to see **continued stabilization work** for Git 2.55, with a focus on **regression fixes**, **documentation updates**, and **late-breaking features**. Key topics to watch:

- **`git history` subcommands**: Both `squash` and `drop` are in `next` and may graduate to `master` if no regressions are reported. User feedback on these experimental commands will shape their future evolution.
- **ODB abstraction**: Patrick Steinhardt’s refdb fix series is the last blocker for the ODB abstraction work, and its resolution will unlock progress toward pluggable backends.
- **Reference management**: The `git refs` subcommands are in `next` and may see follow-up work to improve atomicity or add multi-ref transactions.
- **SHA-256 interoperability**: While not targeted for Git 3.0, the recent milestone may generate renewed discussion about the long-term roadmap for hash algorithm support.
- **Pinned references**: Erik Östlund’s proposal for "pinned references" (a ref + expected OID invariant) remains in early-stage design discussion, and the next month may see a concrete proposal or patches.
- **Git 2.55.0**: The final release is expected in late July, with only regression fixes and late topics expected to graduate before then. Junio’s “What’s cooking” reports will provide regular updates on the stabilization progress.

The June progress reflects the **project’s ongoing commitment to innovation, scalability, and usability**, with a focus on **architectural modernization**, **performance optimization**, and **security hardening**. The coming month will be critical for **stabilizing these efforts** and ensuring they are **robust and well-documented** for the final release.