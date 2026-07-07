# The Git Project Daily Digest
**2026/06/28 (Sunday) – 45 emails, 17 threads**

---

## The day in brief
A **moderate-volume Sunday** with **17 active threads**, dominated by **performance optimizations**, **regression fixes**, and **CI stability work**. The standout developments: **Tian Yuchen’s `paint_down_to_common()` optimization** received final approval, **a regression in `git replay --linearize` was diagnosed**, and **Jeff King’s CI timeout fix** for macOS runners was merged. The day also saw **progress on `git history squash`**, **final readiness for `git branch --set-upstream-to` usability improvements**, and **a lively debate on `USE_NSEC`’s future**.

---

## Notable threads

### `paint_down_to_common()` optimization receives final approval
Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved by Derrick Stolee** after addressing a regression in v3. The series terminates merge-base walks early when one side’s commit queue is exhausted, delivering **100-1000x speedups** for asymmetric queries (e.g., repositories with import grafts). Key changes in v4:
- **Regression fix**: Restored the `min_generation` guard to the generation-monotonicity BUG assertion.
- **Code clarity**: Renamed counters (`parent1_count`, `parent2_count`, `mb_candidate_count`) and consolidated termination conditions in `paint_queue_get()`.
- **Instrumentation**: Added trace2 step counts for observable performance metrics.
The series is now **ready for merging**, with no open technical questions. The optimization is a **major win for `git merge-base` performance** in large repositories, and the test suite robustly verifies correctness across all commit-graph modes.

---

### Regression in `git replay --linearize` diagnosed
Johannes Schindelin reported a **regression in v5 of `git replay --linearize`** that **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact behavior. Schindelin provided a **reproducible test case** (replaying `master~2..master` with `--linearize --onto master~2`) and explained why the parameter is necessary to ensure all replayed commits are linearized into a single topology. The regression is **urgent** and must be fixed before the next release. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled.

---

### CI timeout fix for macOS runners merged
Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` was **merged into `master`**. The series directly targets Apache bug 70131 by:
1. **Increasing Apache’s `Timeout` directive** from 300 to 600 seconds (patch 1/3).
2. **Isolating the expensive "many-tags" test case** into a dedicated repository (patch 2/3).
3. **Packing refs after creating many tags** to reduce `ls-refs` advertisement time (patch 3/3).
The first two patches resolve the immediate CI failures, while the third is a belt-and-braces optimization. Junio C Hamano noted a minor stylistic nit (non-bare repository initialization) but applied the series as-is. The fix is **low-risk** and eliminates a persistent CI flake.

---

### `git history squash` reaches feature-complete status
Harald Nordgren posted **v6 of `git history squash`**, a new subcommand that folds a commit range into its oldest commit while preserving descendant history. The series is now **functionally complete**, with key improvements:
- **Stricter input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **Support for multiple revision arguments**: Allows compound ranges (e.g., `@~3.. ^topic`) to exclude commits on other branches.
- **Expanded documentation**: Clarifies the merge limitation and `--reedit-message` behavior.
The series is **ready for Junio’s final review**, with no open technical questions. Phillip Wood’s usability concerns about `--reedit-message`’s default behavior remain unresolved but do not block merging.

---

### `git branch --set-upstream-to` usability improvements merged
Harald Nordgren’s **2-patch series** improving error messages for common command-line mistakes was **merged into `next`**. The patches:
1. **Detect missing slashes** in `--set-upstream-to` arguments (e.g., `origin main` instead of `origin/main`) and suggest the correct syntax when the remote-tracking ref exists.
2. **Detect slash-as-space mistakes** in `git push` (e.g., `origin/main` instead of `origin main`) and guide users toward the correct syntax.
The series is **well-tested** and aligns with Git’s usability goals. Junio noted a minor design quibble (redundant advice check) but deemed it too small to warrant a rewrite.

---

### `USE_NSEC` debate intensifies
The discussion around D. Ben Knoble’s **Meson `nanosec` option** (mirroring Autotools’ `USE_NSEC`) expanded into a **broader debate** about the knob’s viability. Key developments:
- **Jeff King’s testing** confirmed that modern Linux filesystems (ext4, ext2, XFS, vfat, CIFS, NTFS, FUSE) preserve nanosecond timestamps correctly, obsoleting historical "racy Git" concerns.
- **Interoperability risks** emerged: mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit) could trigger stat-dirty re-reads, degrading performance.
- **Patrick Steinhardt proposed a middle ground**: always compile nanosecond support into Git (when the platform allows it) and expose it via a runtime config knob (e.g., `core.useNsec`), but retain the current build-time default (`USE_NSEC` disabled).
The original patch remains **queued for merging**, but its justification is now in question. The discussion may lead to **deprecation of the build-time knob** in favor of runtime configuration.

---

### Reftable quadratic-time behavior fix reconsidered
Kristofer Karlsson’s **merged fix** for quadratic-time behavior in the reftable backend is now **under reconsideration** due to limited real-world impact. The patch exposes tombstones to iterator bounds checks, reducing runtime from O(n²) to O(n) for bulk ref deletion/re-creation workflows. However, Patrick Steinhardt noted that the fix’s impact is negligible when compaction is enabled, and the added code complexity may not be justified. The patch remains merged for now, but its long-term retention depends on demonstrating **measurable improvement in realistic scenarios**.

---

## In brief

**`git cat-file --batch-command`** -- Pablo Sabater’s **14-patch GSoC series** is **technically complete** and ready for merging. The series introduces a `remote-object-info` command for querying object metadata from remotes, with dynamic format validation and strict protocol v2 enforcement.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **2-line patch** fixing a regression in HTTPS proxy URL validation was **merged into `master`**. The fix restores support for HTTPS proxies, which were incorrectly rejected due to a control-flow error.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

**`greplint.pl` regression** -- Junio C Hamano identified a **critical oversight** in the `greplint.pl` series: the existing `test_grep` implementation already checks for file existence, yet the bug in `t3420-rebase-autostash.sh` persists. The discussion now centers on **debugging why the safeguard fails** rather than adding new tooling.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue.
- **`git history squash`**: Junio’s final review pending.
- **`USE_NSEC` debate**: May lead to deprecation of the build-time knob in favor of runtime configuration.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.