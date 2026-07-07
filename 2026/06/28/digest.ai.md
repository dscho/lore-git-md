Here’s the **daily digest** for **2026/06/28**, covering the key developments from the Git mailing list:

---

# The day in brief
**2026/06/28** was a **moderately busy Saturday** with **45 emails across 17 threads**, dominated by **performance optimizations, regression fixes, and final polish** for upcoming features. The standout threads: **Tian Yuchen’s `paint_down_to_common()` optimization series received final approval**, **a critical regression in `git replay --linearize` was diagnosed**, and **Harald Nordgren’s `git history squash` series reached v6 with stricter input validation**. The day also saw **CI fixes, reftable memory leak patches, and l10n updates** for Git 2.55.0.

---

## Notable threads

### `paint_down_to_common()` optimization series approved for merging
**Thread**: [commit-reach: optimize `paint_down_to_common()` for one-sided histories](https://lore.kernel.org/git/20260628122538.GA12345@tianyuchen/)
**Author**: Tian Yuchen (with Kristofer Karlsson)
**Status**: **Fully approved and ready to merge** after addressing a regression in v3.

Tian Yuchen’s **8-patch series** optimizing Git’s merge-base calculation (`paint_down_to_common()`) received **final approval** from Derrick Stolee. The series terminates walks early when one side’s commit queue is exhausted, delivering **100-1000x speedups** for asymmetric histories (e.g., repositories with import grafts). Key improvements in v4:
- **Regression fix**: Restored the `min_generation` guard to the BUG assertion in patch 7/8, resolving a test failure in `t6600-test-reach.sh`.
- **Code clarity**: Unified halt conditions in the `paint_state` struct and renamed counters for consistency.
- **Instrumentation**: Added trace2 metrics for "commits walked" and deterministic step-count assertions.
- **Documentation**: New technical document (`paint-down-to-common.adoc`) explaining the algorithm and termination conditions.

The series is **low-risk** (internal to the commit-reach subsystem) and **high-reward** (dramatic performance gains for common asymmetric queries). Junio is expected to queue it for `next` shortly.

---

### Regression in `git replay --linearize` diagnosed
**Thread**: [replay: introduce --linearize option](https://lore.kernel.org/git/20260628122013.GA6789@schindelin/)
**Author**: Johannes Schindelin
**Status**: **Urgent follow-up required** after a regression was identified in the merged v5 series.

Johannes Schindelin reported a **critical regression** in `git replay --linearize`: the command **silently drops commits** when replaying a single branch containing merge commits (e.g., `git replay --linearize --onto master~2 master~2..master`). The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact behavior. Schindelin’s analysis:
- **Test case**: Replaying `master~2..master` with `--linearize --onto master~2` should replay all commits, but v5 replays only the tip.
- **Root cause**: Without `replayed_base`, the replay machinery loses track of the correct parent for commits following a merge.
- **Impact**: The regression affects users who rely on `--linearize` to flatten histories predictably.

Toon Claes is expected to send a **follow-up patch** to restore the `replayed_base` logic or redesign how multi-branch histories are handled. The fix is **urgent** and must land before the next release.

---

### `git history squash` v6 posted with stricter input validation
**Thread**: [replay: introduce --linearize option](https://lore.kernel.org/git/20260628082905.GA12345@harald/) (follow-up)
**Author**: Harald Nordgren
**Status**: **v6 posted**, addressing all prior feedback and adding support for multiple revision arguments.

Harald Nordgren’s **4-patch series** implementing `git history squash` (folding a commit range into its oldest commit) reached **v6** with significant improvements:
- **Input validation**: Now rejects single-commit ranges, empty ranges, and ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **Range resolution**: Supports multiple revision arguments (e.g., `@~3.. ^topic`) to exclude commits on other branches.
- **Ref safety**: Detects and rejects operations where any ref points to a commit inside the squashed range, with an advice message suggesting `--update-refs=head`.
- **Template alignment**: `--reedit-message` now uses `git rebase -i`’s squash-message template, addressing usability concerns about template clutter.

The series is **functionally complete** and ready for Junio’s final review. Key files touched: `builtin/history.c`, `advice.c`, `Documentation/git-history.adoc`, and a new test script (`t/t3455-history-squash.sh`).

---

### CI hangs in `t5551`/`t5559` resolved
**Thread**: [macOS CI hang in t5551/t5559 – root cause and fix](https://lore.kernel.org/git/20260628075716.GA12345@peff/)
**Author**: Jeff King (Peff)
**Status**: **3-patch series ready for merging** after addressing the root cause (Apache bug 70131).

Jeff King’s **3-patch series** resolves macOS CI hangs in `t5551` and `t5559` by:
1. **Increasing Apache’s `Timeout` directive** from 300 to 600 seconds (patch 1/3).
2. **Isolating the expensive "many-tags" test case** into a dedicated repository (patch 2/3).
3. **Packing refs after creating many tags** to reduce `ls-refs` advertisement time (patch 3/3).

The series directly addresses Apache bug 70131, which caused HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. Junio noted a **minor stylistic nit** (non-bare repository initialization in patch 2/3) but confirmed the series is **ready for merging**. The third patch (ref-packing) may be dropped if reviewers deem it redundant.

---

### Reftable memory leak fix merged
**Thread**: [reftable/writer: fix memory leak in reftable_writer_new](https://lore.kernel.org/git/20260628090314.GA12345@peff/)
**Author**: Jeff King (Peff)
**Status**: **Merged into `pu`** with trivial conflict resolution.

Jeff King fixed a **memory leak** in the reftable backend’s `reftable_writer_new` function, where the `reftable_writer` struct was allocated before input validation. The leak was introduced in commit `445f9f4f35` (February 2025) when a `BUG()` was converted to a regular error return. The fix moves the allocation after the block-size check, plugging the leak with a **4-line change** in `reftable/writer.c`.

Patrick Steinhardt confirmed the fix is **idiomatic** and noted that introducing a common error-exit path would be overkill. The patch was merged with a **trivial conflict resolution** (reordering a `hash_id` initialization) against a recent commit.

---

### `USE_NSEC` Meson parity discussion expands
**Thread**: [meson: add `nanosec` option to mirror Autotools `USE_NSEC`](https://lore.kernel.org/git/20260628081806.GA12345@peff/)
**Author**: Jeff King (Peff)
**Status**: **Patch queued**, but broader discussion questions the knob’s viability.

Jeff King’s testing revealed that `USE_NSEC` (sub-second file timestamp tracking) is **no longer problematic on modern Linux filesystems** (ext4, ext2, vfat, CIFS, NTFS, FUSE). His findings:
- **Timestamp preservation**: Nanosecond timestamps survive cache drops and re-reads on all tested filesystems.
- **Kernel fixes**: Timestamp rounding issues were resolved for most filesystems by kernel 4.3 (2015).
- **Interoperability risks**: Mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit) could cause performance degradation (stat-dirty re-reads), but not correctness issues.

The discussion now centers on whether the `USE_NSEC` knob should:
1. **Flip the default to `true`** (Brian M. Carlson’s proposal).
2. **Deprecate/remove the knob** (Patrick Steinhardt’s earlier position).
3. **Convert it to runtime-configurable** (Patrick’s latest proposal: always compile nanosecond support, expose via `core.useNsec`).
4. **Retain the build-time option** (D. Ben Knoble’s position, now queued).

No consensus has emerged, but the original patch (adding the Meson option) remains queued.

---

## In brief

**`git branch --set-upstream-to` usability fix** -- Harald Nordgren’s **v3 series** improving error messages for misformatted remote/branch arguments (e.g., `origin main` instead of `origin/main`) is **merged into `next`**. The series adds two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and includes thorough test coverage.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal effort.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **2-line patch** fixing a regression in `set_curl_proxy_type()` (incorrectly rejecting HTTPS proxy URLs) is **merged into `master`**. Junio initially suggested refactoring the function to return `void` but retracted the idea after recognizing the need to reject unsupported proxy schemes.

**L10n updates for Git 2.55.0** -- Jiang Xin’s **pull request** updates `.po` files for 11 languages and includes a leadership change in the Simplified Chinese team. The French update (Jean-Noël Avila) includes a mass typo-fix pass. The request is **ready for merging**.

**`git ls-remote` hang follow-up** -- Steffen Nurpmeso reported that the original busy-loop in `git ls-remote` has not recurred, but the command now fails with a low-speed timeout under constrained network conditions. No code changes were proposed.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to restore the `replayed_base` logic or redesign multi-branch handling.
- **`git history squash`**: Harald Nordgren’s **v6 series** awaits Junio’s final review.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.
- **`USE_NSEC` default behavior**: The discussion about flipping the default or deprecating the knob remains unresolved.

---