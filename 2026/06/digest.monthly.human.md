# Git Mailing List Monthly Digest -- 2026 June

## The period in brief

June 2026 was an **exceptionally productive month** for the Git project, with **2,184 emails across 591 threads** spanning all seven days of each week. Traffic was **heavy and technically substantive**, featuring the **completion of several multi-year efforts**, **major architectural refactoring**, and **significant performance and usability improvements**. The most important developments: **the ODB abstraction work reached a major milestone**, **`git history` subcommands (`squash`, `drop`) landed in `next`**, **`git branch --delete-merged` completed its 18-iteration review cycle**, and **Git v2.55.0-rc1 was released**, marking the start of the stabilization period for the next major release. Readers who followed the weekly digests closely will find some redundancy here; those who skipped them should focus on the **history editing tools**, **reference management consolidation**, **ODB abstraction progress**, and **security-hardening efforts**—these are the developments that will shape Git’s evolution in the coming months and years.

---

## Key developments

### ODB abstraction reaches major architectural milestone

Patrick Steinhardt’s **multi-year effort to abstract Git’s object storage layer** saw **decisive progress** in June, with **two major series landing in `next`** and a third reaching technical completion. The **18-patch series refactoring loose object handling** was merged, converting the loose object source into a standalone `struct odb_source` implementation while maintaining identical runtime behavior. This was followed by a **16-patch series completing the packed object storage conversion**, making packed storage a proper pluggable backend. The changes touch 41 files and represent a **foundational step toward pluggable storage backends**, enabling future alternatives to loose objects and packfiles.

The **17-patch series converting packed object storage to use `struct odb_source`** reached technical completion, with all callbacks wired up and the MIDX interface refactored. The series is **mechanically complete** but remains blocked by **unresolved refdb architectural issues** (memory leaks, duplicate refdb creation, initialization ordering) that Patrick is addressing in a separate **11-patch fix series**. Junio Hamano acknowledged the series as **ready for integration once the refdb issues are resolved**, and the work represents the **final large-scale ODB restructuring** Patrick plans to undertake. The mechanical changes are uncontroversial, but the architectural dependencies highlight the complexity of modernizing Git’s storage layer while maintaining backward compatibility.

During implementation, Patrick uncovered deeper **refdb architectural issues** that will require follow-up work to resolve memory leaks and initialization problems. The ODB abstraction effort is now **poised to enable pluggable backends**, including cloud storage and alternative formats, and it sets the stage for future improvements in Git’s storage layer.

---

### `git history` subcommands mature: `squash` and `drop` reach key milestones

The **experimental `git history` command**, introduced in Git 2.53 as a vehicle for more flexible history editing, saw **two major subcommands reach critical milestones** in June. Harald Nordgren’s **`git history squash`**—a tool to fold a commit range into its oldest commit while preserving descendant history—**landed in `next`** after 13 iterations and a decisive pivot from a proposed `git rebase --squash` option. The implementation avoids the efficiency pitfalls of rebase-based approaches (repeated conflict resolution) by computing a single three-way merge of the entire range against upstream, and it handles edge cases like fixup-commit sequences and merge commits correctly. The design also resolved a key safety concern: by default, the command **rejects operations where refs point into the squashed range**, with an `advice()` message directing users to `--update-refs=head` if they want to retarget those refs to the squashed commit.

Separately, Patrick Steinhardt’s **`git history drop`**—a subcommand to remove a commit while replaying its descendants—**completed its technical implementation** and was introduced in Junio’s “What’s cooking” report. The series modernized Git’s reset machinery as part of the effort and includes **537 lines of test coverage** for safety mechanisms. Both subcommands are part of a broader effort to provide **more flexible and safer alternatives to `git rebase -i` and `git filter-branch`**, and they reflect the project’s increasing comfort with experimental commands that can evolve based on user feedback. The subcommands are now **available in `next`** and may graduate to `master` in the coming weeks if no regressions are reported.

---

### `git branch --delete-merged` reaches technical completion

Harald Nordgren’s **18-iteration series** adding `git branch --delete-merged`—a tool for safe automated cleanup of merged local branches—**reached technical completion** in June, with all design and implementation feedback addressed. The command deletes branches that have been merged into a specified commit (defaulting to `HEAD`), with extensive safety mechanisms: it **aborts if the branch is checked out**, **warns if the branch has been pushed to a remote with divergent history**, and **allows per-branch opt-out via config** (`branch.<name>.deleteMerged=false`). The series also includes a **`--dry-run` preview** and a **`--force` override** for the safety checks.

The final design hurdle—**stacked-branch safety**—was resolved with a pragmatic approach: the command **aborts by default** if the branch being deleted is used as an upstream for another unmerged branch (e.g., `feature2` stacks on `feature1`), with a clear error message. This avoids the complexity of automatic retargeting (e.g., recursion in chains like `b1→b2→b3`) while preserving backward compatibility for future enhancements. The series is now **ready for final review** and likely to graduate to `next` in the coming weeks. The feature represents a **significant quality-of-life improvement** for branch management, particularly in workflows involving frequent branch cleanup.

---

### Reference management consolidation: `git refs` subcommands land

Patrick Steinhardt’s **five-patch series** adding reference-writing subcommands (`create`, `delete`, `update`, `rename`) to `git refs` **landed in `next`** in June, consolidating functionality previously scattered across `git-update-ref` and `git-symbolic-ref`. The series improves discoverability and reduces fragmentation in Git’s reference management interface, particularly for scripting and automation.

The design debate about **atomicity vs. backward compatibility** was resolved in favor of retaining `update`’s implicit creation behavior (for script compatibility) while adding a dedicated `create` subcommand for explicit atomic creation. This compromise reflects the project’s preference for **gradual evolution** over breaking changes, even when the latter might offer cleaner semantics. The series also aligns with Patrick’s broader ref backend modernization efforts, including the ongoing ODB abstraction work, and it sets the stage for future improvements like **atomic multi-ref transactions**.

---

### Git v2.55.0-rc1 released, marking the start of the stabilization period

Junio C Hamano **released Git v2.55.0-rc1** on June 17, incorporating **460 non-merge commits from 82 contributors**. The release includes **new features** like parallel hook execution, the `git format-rev` builtin, and fsmonitor support for Linux, along with **significant performance optimizations** and **internal refactoring** (e.g., ODB abstraction, `the_repository` removal, Rust integration). The sheer volume of changes—including ongoing efforts like the ODB abstraction and history editing tools—makes this a **significant release**, and the focus now shifts to **regression fixes and documentation updates**.

The -rc1 tarballs are available for testing, and the stabilization period will continue until the final release, expected in late July. The release notes highlight the **experimental status** of `git history` and the **performance improvements** in `git log --graph` and `git branch --remotes`, among other changes. The release also includes **security-hardening efforts**, such as the sideband sanitization series addressing CVE-2024-32002 and CVE-2024-52005, which will now ship in Git 2.55 rather than waiting for Git 3.0.

---

### Security-hardening efforts advance on multiple fronts

June saw **significant progress on security-hardening efforts** across several subsystems. Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries (e.g., object sizes) reached **final readiness**, with all substantive concerns resolved. The series introduces a new `remote-object-info` command that lets clients request metadata for up to 10,000 objects in a single command, reducing network overhead for partial clone workflows. The series is **security-hardened**, with strict protocol v2 enforcement, input validation, and dynamic capability-based validation of format placeholders. The only unresolved question is **error-handling philosophy**: should the client fail explicitly when metadata is missing, or continue silently (matching local `git cat-file` behavior)? The series is **ready for merging** and represents a **major improvement** for large-repository workflows.

Separately, Patrick Steinhardt’s **11-patch series hardening the reftable backend** against corrupted files was **fully reviewed and ready for `next`**. The series fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in the reftable parser, and adds a **libFuzzer-based fuzzer** to prevent regressions. The fixes are **internal-only** (no user-facing changes) and critical for the reftable backend’s stability. The fuzzing infrastructure is a **long-term win** for security, and the series is expected to graduate to `master` shortly.

Junio Hamano also announced that the **sideband sanitization series** addressing CVE-2024-32002 and CVE-2024-52005 will now ship in Git 2.55 rather than waiting for Git 3.0. The change removes the transitional compatibility layer after successful production validation in Git for Windows and Red Hat environments. The implementation prevents terminal state corruption and input buffer injection attacks while preserving legitimate ANSI color sequences used by pre-receive hooks.

---

### Performance optimizations show dramatic gains

June featured **multiple performance optimizations** that demonstrated **dramatic speedups** across several commands. Tian Yuchen’s **8-patch series optimizing `paint_down_to_common()`** for one-sided histories was **fully approved and merged**, delivering **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts). The series terminates early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories. The optimization targets a known bottleneck in repositories with asymmetric histories (e.g., import grafts or shallow histories), and the series includes **trace2 instrumentation**, **dead-code removal**, and a new technical document (`paint-down-to-common.adoc`).

Other notable performance improvements included:
- **Tamir Duberstein’s ref-filter optimizations**, which fixed a regression causing ~25x slowdowns when listing refs with many loose refs.
- **`git describe` optimizations** by Tamir Duberstein, which showed **17x speedups** (172ms→10ms) by limiting ref iteration to tags when `--all` isn’t specified.
- **Kristofer Karlsson’s merge-base optimization**, which demonstrated **300-1000x speedups** (4.85s → 6ms) for monorepo scenarios through early termination.
- **René Scharfe’s `git cat-file --batch-check` formatting optimization**, which delivered **3-7% faster object queries**.

These optimizations reflect the project’s **ongoing focus on scalability**, particularly for large repositories and monorepos.

---

### `git replay --linearize` post-merge regression: silent commit dropping

A **post-merge regression** in `git replay --linearize` (v5) was reported in June, where the command **silently drops commits** when replaying a single branch containing merge commits. In a real-world example (`master~2..master`), only the tip commit is replayed, and the “Git 2.55-rc2” commit is lost. The regression stems from the removal of the `replayed_base` logic in `pick_regular_commit()`. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled. This is a **critical regression** that needs urgent attention before the next release, as it affects the correctness of history editing operations.

---

### Windows-specific improvements and large-object support

June saw **significant progress on Windows-specific improvements**, including the upstreaming of long-standing patches from Git for Windows. Johannes Schindelin proposed upstreaming two patches: one introducing gentler process termination using thread injection to call `ExitProcess()`, and another adding proper SIGINT handling via `ConsoleCtrlHandler`. These address long-standing pain points where Windows’ lack of Unix-style signals prevented proper cleanup. The patches have extensive production validation from Git for Windows and are **ready for review**.

Separately, Philip Oakley’s **4.5-year-old series addressing >4GB object support on Windows** was revived, converting object header length handling from `unsigned long` to `size_t`. Johannes Schindelin’s **8-patch series** converting `unsigned long` to `size_t` for object sizes on Windows (where the two types differ) received **Junio’s ack** for its delta-handling patch. The series, now **rebased and conflict-free**, addresses 4GB+ object support by systematically updating object-file, ODB, and hash algorithm code paths. All production changes have been validated in Git for Windows, and the series is **poised for `next`** pending final review.

---

### `git repack --geometric --cruft` RFC: correctness flaw diagnosed

Taylor Blau’s **10-patch RFC** to combine `--geometric` and `--cruft` repack modes hit a **correctness snag** in June: Junio identified a flaw in the two-phase traversal logic of `--stdin-packs=follow-reachable`, where **unreachable tags and objects** may be incorrectly retained. The current implementation marks *all* tag objects in included packs as reachable, even if they’re not referenced by any ref. Taylor proposed reversing the traversal order (walk from refs first, then mark objects in included packs), but no concrete fix has been submitted. The series remains in **RFC limbo** until this is resolved, but it represents a **high-impact workflow improvement** for large repositories.

---

## In brief

**`git history squash` v6** -- Harald Nordgren’s series to fold commit ranges into a single commit is **functionally complete**, with stricter input validation, support for multiple revision arguments, and the `--reedit-message` flag. The series is ready for Junio’s final review.

**`git log -L` range-scoped diff stat** -- Karthik Nayak’s 7-patch series extending `git log -L` to support `--stat`, `--check`, and `-G` pickaxe operations is **complete and uncontroversial**. The series adds range-scoped diff stat formats and whitespace checking.

**`excludes_file` libification** -- Tian Yuchen’s patch moving the global `excludes_file` into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series is part of the **libification effort** to eliminate global state.

**`git history --reword` file leak fix** -- Junio’s **bugfix patch** addresses a file stream leak in `git history --reword` that could cause problems on Windows. The fix consolidates file handling and is **merge-ready**.

**`--track=fetch` workflow debate** -- Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on **philosophical concerns** about network operations during checkout. Junio’s reservations about "blind updates" have not been addressed.

**CI hangs in `t5551`/`t5559`** -- Jeff King’s 3-patch series increases Apache’s `Timeout` directive to 600 seconds and isolates the expensive “many-tags” test case. The fix is **ready for merging**.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass and is **ready for merging**.

**`greplint.pl` exposes test-suite fragility** -- Junio demands an audit of all `# lint-ok:` comments to ensure they do not hide latent test bugs. The discussion highlights a tension between automation and diagnostic rigor.

**`git ls-files` optimization** -- Tamir Duberstein’s patch filtering pathspecs before `lstat` operations shows **50x speedup** (60.7s → 1.06s) in large repositories.

**Subcommand autocorrection** -- Jiamu Sun’s typo correction for Git subcommands like `git remote` was merged to 'seen' pending final style fixes.

**Test infrastructure hardening** -- Patrick Steinhardt’s series enforces valid TAP output in tests and standardizes CI environments across GitLab and GitHub.

**Worktree metadata tracking** -- Norbert Kiesel’s series added creation timestamps and free-form descriptions to worktrees with sorting options.

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

**`git pack-objects` gains full `--delta-islands`/`--path-walk` compatibility** -- Taylor Blau’s four-patch v3 series resolved the long-standing limitation where `--path-walk` could not leverage bitmaps or delta-islands. The implementation enables sequential compatibility: bitmaps accelerate object counting (with path-walk as a one-way fallback), while delta-islands are respected during path-walk traversal via tree-depth recording.

**SHA-256 interoperability hits a milestone** -- Brian M. Carlson announced that **SHA-1/SHA-256 interoperability** now passes its test suite, with core functionality (shallow clones, submodules, partial clones) working. However, **critical limitations** remain: packfile URIs and promisor remotes are unsupported due to architectural constraints (e.g., unmapped objects in disconnected packs).

**`Assisted-by:` trailer proposal divides contributors** -- A **controversial RFC** from Marius Spix proposed adopting an `Assisted-by:` trailer for AI-assisted commits, mirroring the Linux kernel’s recent policy. Kristoffer Haugsbakk **pushed back hard**, arguing that Git has historically rejected project-specific trailers (citing a 2013 `Fixes:` rejection) and that the proposal would force Git to make policy decisions (e.g., auto-propagating "taint" during merges).

**MIDX incremental repack regression fixed** -- Taylor Blau’s **3-patch series** fixed a regression in `git multi-pack-index write --incremental --base`, restoring reachability closure for bitmaps. The regression (introduced in Git 2.55-rc1) caused the command to ignore custom bases, breaking incremental repacking.

**Commit-reach optimization series lands with 500× speedup** -- Tian Yuchen’s six-patch series optimizing `paint_down_to_common()` for one-sided histories **landed**, introducing an early-exit condition that terminates the merge-base walk when one side’s queue is exhausted. The optimization targets a known bottleneck in repositories with asymmetric histories (e.g., import grafts or shallow histories), with benchmarks showing **100–1000× speedups** for large one-sided histories (2.6M commits).

**Worktree diff performance investigation yields minimal root-cause fix** -- A performance investigation into `git diff` slowdowns in secondary worktrees (223.3ms vs 3.4ms in the main worktree) culminated in Jeff King’s **minimal patch** to remove the `#ifdef USE_NSEC` guard in `read-cache.c:is_racy_stat()`. The change enables nanosecond-aware timestamp comparisons unconditionally, eliminating racy entries at the source by leveraging existing nanosecond precision infrastructure.

**Shell completion dotfile hiding blocked on structural overhaul** -- Zakariyah Ali’s v3 series aligning Git’s path completion with shell conventions (hiding dotfiles unless explicitly requested) remains **blocked** on Junio’s request for a cleaner two-patch progression: (1) refactor the AWK script in `__git_index_files` for readability without changing behavior, and (2) implement dotfile hiding on top of that refactored base.

**`git repo info` path formatting series ready for pickup** -- A GSoC participant’s three-patch v7 series introducing standardized path formatting for repository information is now **ready for Junio’s pickup**. The series adds `append_formatted_path()` to consolidate duplicated logic in `rev-parse` and exposes the repository’s Git directory and common directory paths in both absolute and relative formats for scriptability.

**Ref backend refactoring advances toward final review** -- Patrick Steinhardt’s 10-part v4 series modernizing Git’s reference backend infrastructure has addressed Jeff King’s architectural concerns about recursive initialization during ref store setup. The series introduces a targeted `ignore_refs` flag in `struct config_options` to selectively disable "onbranch" config includes during ref store initialization, preventing recursive ref store access while preserving other include conditions.

**Documentation update on patch reroll cadence ready for merging** -- Weijie Yuan’s single-patch v3 series updating `SubmittingPatches` to codify implicit norms around patch reroll cadence is now **uncontested and ready for Junio to pick up**. The patch advises contributors to batch rerolls (no more than one per day by default) and signal which parts of a series will become obsolete when substantial rework is needed.

**Build system quietness patches merged-ready for gitk and git-gui** -- Harald Nordgren’s two-patch series aligning gitk and git-gui’s translation catalog generation with core Git’s quiet build conventions is now **procedurally ready for merging**. The patches remove `--statistics` from `msgfmt` invocations and conditionally suppress "Generating catalog" messages when `make -s` is used.

**`ignore_case` libification merged** -- Justin Tobler and Tian Yuchen’s two-patch series moving the global `ignore_case` variable into `struct repo_config_values` was **accepted by Junio**. The series introduces a `repo_ignore_case()` getter and migrates all usage sites, eliminating a global and tying the configuration value to the repository instance it was read from.

---

## Looking ahead

The next month is likely to see **continued stabilization work** for Git 2.55, with a focus on **regression fixes** and **documentation updates**. Key topics to watch:

- **`git history` subcommands**: Both `squash` and `drop` are in `next` and may graduate to `master` if no regressions are reported. User feedback on these experimental commands will shape their future evolution.
- **ODB abstraction**: Patrick Steinhardt’s refdb fix series is the last blocker for the ODB abstraction work, and its resolution will unlock progress toward pluggable backends.
- **Reference management**: The `git refs` subcommands are in `next` and may see follow-up work to improve atomicity or add multi-ref transactions.
- **SHA-256 interoperability**: While not targeted for Git 3.0, the recent milestone may generate renewed discussion about the long-term roadmap for hash algorithm support.
- **Pinned references**: Erik Östlund’s proposal for "pinned references" (a ref + expected OID invariant) remains in early-stage design discussion, and the next month may see a concrete proposal or patches.
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue before the next release.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw. The series is a **high-impact workflow improvement** for large repositories.
- **`--track=fetch`**: The philosophical debate over network operations during checkout remains unresolved. Junio’s final decision is pending.
- **Git 2.55.0**: The release is imminent, with only regression fixes and late topics expected to graduate before the final release. Junio’s “What’s cooking” report (#09) signals deep feature-freeze.