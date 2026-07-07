# The Git Project Daily Digest
**2026/06/28 (UTC)**

## The day in brief
A **moderately busy Sunday** with **45 emails across 17 threads**, dominated by **performance optimizations**, **regression fixes**, and **final polish** on long-running series. The standout developments: **Tian Yuchen’s `paint_down_to_common()` optimization** received final approval, **Harald Nordgren’s `git history squash` series** reached v6 with stricter input validation, and **Jeff King’s CI hang fix** for `t5551`/`t5559` was posted. A **post-merge regression in `git replay --linearize`** resurfaced, requiring urgent attention.

---

## Notable threads

### `paint_down_to_common()` optimization receives final approval
Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved by Derrick Stolee** after addressing a regression in v3. The series terminates merge-base walks early when one side’s commit queue is exhausted, delivering **100-1000x speedups** for asymmetric queries (e.g., repositories with import grafts). Key improvements in v4:
- **Regression fix**: Restored the `min_generation` guard to the generation-monotonicity BUG assertion.
- **Code clarity**: Renamed counters (`parent1_count`, `parent2_count`, `mb_candidate_count`) and consolidated termination conditions in `paint_queue_get()`.
- **Instrumentation**: Added trace2 metrics for "commits walked" and deterministic step-count assertions.
The series is **ready for merging** and represents a **major performance win** for `git merge-base` in asymmetric histories.

---

### `git history squash` reaches v6 with stricter input validation
Harald Nordgren’s **4-patch series** implementing `git history squash` (folding a commit range into its oldest commit) reached **v6** with **stricter input validation** and support for multiple revision arguments (e.g., `@~3.. ^topic`). The series now:
- **Rejects single-commit ranges** (e.g., `@^!`) with "nothing to squash."
- **Rejects ranges with multiple bases** (e.g., a side branch forked before the range).
- **Supports compound ranges** (e.g., `@~3.. ^topic`) to exclude commits on other branches.
- **Improves test coverage** (550+ lines) for merge topologies and interior ref detection.
The series is **functionally complete** and ready for Junio’s final review. A **philosophical debate** over whether `--reedit-message` should be the default remains unresolved.

---

### CI hang fix for `t5551`/`t5559` posted
Jeff King (Peff) sent a **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` caused by Apache’s 300-second timeout during `ls-refs` advertisements of 100,000 loose refs. The series:
1. **Increases Apache’s `Timeout` directive to 600 seconds** (directly addressing the root cause).
2. **Isolates the expensive "many-tags" test case** into a dedicated repository.
3. **Packs refs after creating many tags** to reduce `ls-refs` advertisement time.
The first patch alone resolves the CI hangs, while the latter two are optimizations. The series is **low-risk and ready for merging**.

---

### `git replay --linearize` regression resurfaces
Johannes Schindelin reported a **post-merge regression** in `git replay --linearize`: the command **silently drops commits** when replaying a single branch containing merge commits (e.g., `master~2..master` with `--linearize --onto master~2`). The issue stems from the removal of the `replayed_base` parameter in v5, which was necessary to prevent commit dropping. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled. The regression is **urgent** and must be fixed before the next release.

---

### `USE_NSEC` Meson parity debate expands
The discussion around D. Ben Knoble’s patch adding a `nanosec` Meson option for `USE_NSEC` expanded into a **broader debate** about the knob’s viability. Jeff King’s testing confirmed that **modern Linux filesystems (ext4, ext2, XFS, vfat, CIFS, NTFS, FUSE) preserve nanosecond timestamps correctly**, obsoleting historical "racy Git" concerns. However, **interoperability risks** emerge when mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit), which could trigger stat-dirty re-reads. Patrick Steinhardt proposed an **incremental path forward**: always compile nanosecond support into Git (when the platform allows it) and expose it via a runtime config knob (e.g., `core.useNsec`), but retain the current build-time default (`USE_NSEC` disabled). The proposal sidesteps auto-detection challenges but leaves the default value for the config knob unresolved.

---

### `excludes_file` libification guardrail resolved
Tian Yuchen and Junio C Hamano resolved the final design question for the `excludes_file` migration series: **remove the redundant `repo->initialized` check from `repo_excludes_file()`** and rely entirely on `repo_config_values()` to enforce initialization. The series is now **ready for `next`**, though CI failures are expected as the stricter design surfaces previously hidden invalid callers. The resolution sets a precedent for future repository-aware getters (e.g., `attributes_file` migration).

---

## In brief

**`git cat-file --batch-command`** -- Pablo Sabater’s **14-patch GSoC series** is technically complete and ready for merging. The series introduces a `remote-object-info` command for querying object metadata (e.g., size) from remotes without full downloads. A **philosophical debate** over silent failures versus explicit errors remains unresolved.

**`git branch --set-upstream-to` usability fix** -- Harald Nordgren’s **2-patch series** improving error messages for common command-line mistakes (e.g., `git branch --set-upstream-to=origin main`) was **merged into `next`**. The series adds two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and includes thorough test coverage.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions was **merged (gitk) or superseded (git-gui)**. The changes will be proposed for upstream Git inclusion in the coming weeks.

**`greplint.pl` regression** -- Junio C Hamano identified a **critical oversight** in the `greplint.pl` series: the proposed `test_grep` enhancement to error on missing files is redundant because `test_grep` already validates file existence. The discussion now focuses on **debugging why the existing safeguard fails** in `t3420-rebase-autostash.sh`.

**Reftable memory leak fix** -- Jeff King’s **bugfix patch** for a memory leak in `reftable_writer_new` was **merged into `pu`** with a trivial conflict resolution. The fix moves the allocation after input validation to prevent leaks on invalid block sizes.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **bugfix patch** for a regression rejecting HTTPS proxy URLs was **merged into `master`**. The fix adds a missing `return 0` in `set_curl_proxy_type()` to prevent fall-through to an error return.

**French translation update** -- Jiang Xin’s **l10n pull request** for Git 2.55.0 updates `.po` files for 11 languages and includes a mass typo-fix pass for French. The pull request is **ready for merging**.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **`git history squash`**: Harald Nordgren’s **v6 series** is ready for Junio’s final review.
- **`USE_NSEC` runtime configuration**: Patrick Steinhardt’s proposal to make `USE_NSEC` runtime-configurable requires further discussion.
- **`excludes_file` migration**: Tian Yuchen’s series is ready for `next` after resolving the guardrail debate.