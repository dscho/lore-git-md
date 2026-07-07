# The Git Project Daily Digest
**2026/06/28 (UTC)**

A **moderately busy Sunday** with **45 emails across 17 threads**, dominated by **performance optimizations**, **regression fixes**, and **final readiness reviews** for long-running series. The standout items: **Tian Yuchen’s `paint_down_to_common()` optimization** received final approval, **Harald Nordgren’s `git history squash` series** reached feature-complete status, and a **regression in `git replay --linearize`** resurfaced, requiring urgent attention. The day also saw **CI/platform-compatibility fixes** and **build system discussions** gain momentum.

---

## Notable threads

### `paint_down_to_common()` optimization receives final approval
**Thread**: [commit-reach: optimize `paint_down_to_common()` for one-sided histories](https://lore.kernel.org/git/20260628122538.17173-1-kristoffer.karlsson@sonymobile.com/)
**Author**: Kristofer Karlsson (via Tian Yuchen)

Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved by Derrick Stolee** after addressing a **regression in v3** (an unconditional BUG assertion). The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Key improvements in v4:
- **Regression fix**: Restored the `min_generation` guard to the BUG assertion in patch 7/8.
- **Code clarity**: Unified halt conditions in the `paint_state` struct.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).

The optimization delivers **100-1000x speedups** for asymmetric queries (e.g., `git merge-base --all` across a 2.6M-commit monorepo drops from **3.67s to 5ms**). The series is **ready for merging** and represents a **major win for `git merge-base` performance** in asymmetric histories.

---

### `git history squash` reaches feature-complete status
**Thread**: [PATCH v6 0/4] replay: introduce --linearize option](https://lore.kernel.org/git/20260628082905.12345-1-haraldnordgren@gmail.com/)
**Author**: Harald Nordgren

Harald Nordgren’s **v6 series** implementing `git history squash` (folding a commit range into its oldest commit while preserving descendant history) is now **feature-complete** and ready for Junio’s final review. Key improvements in v6:
- **Stricter input validation**: Rejects single-commit ranges, empty ranges, and ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`.
- **Support for multiple revision arguments**: Accepts compound ranges (e.g., `@~3.. ^topic`) to exclude commits on other branches.
- **Expanded documentation**: Clarifies the merge limitation and `--reedit-message` behavior.
- **Test coverage**: 747-line test suite covering range parsing, merge handling, ref safety, and `--reedit-message` template formatting.

The series addresses all prior feedback, including **Junio’s CLI/documentation style corrections** and **Phillip Wood’s usability-driven template polish**. The only remaining debate is whether `--reedit-message` (or `--edit`) should be the default, but this is not a blocker.

---

### Regression in `git replay --linearize` resurfaces
**Thread**: [PATCH 0/3] replay: introduce --linearize option](https://lore.kernel.org/git/20260628122013.5678-1-Johannes.Schindelin@gmx.de/)
**Author**: Johannes Schindelin

A **regression in `git replay --linearize`** (merged in v5) was reported by Johannes Schindelin: the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact behavior. Schindelin’s test case:
```sh
git replay --linearize --onto master~2 master~2..master
```
now replays only the tip commit ("Git 2.55-rc2") instead of all commits in the range. The regression is **urgent** and must be fixed before the next release. Toon Claes is expected to send a follow-up patch to restore the `replayed_base` logic or redesign how multi-branch histories are handled.

---

### CI/platform-compatibility fixes for macOS hangs
**Thread**: [macOS CI hang in t5551/t5559 – root cause and fix](https://lore.kernel.org/git/20260628075716.GA12345@peff.net/)
**Author**: Jeff King

Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` is **ready for merging**. The series targets Apache bug 70131, which causes HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. Key changes:
1. **Patch 1/3**: Increases Apache’s `Timeout` directive from 300 to 600 seconds.
2. **Patch 2/3**: Isolates the expensive "many-tags" test case into a dedicated repository.
3. **Patch 3/3**: Packs refs after creation to reduce `ls-refs` advertisement time.

The series is **low-risk** and directly addresses the root cause. Junio’s review of patch 3/3 suggests it may be redundant given the other optimizations, but the first two patches alone resolve the CI failures.

---

### `USE_NSEC` Meson parity discussion expands
**Thread**: [PATCH] meson: add `nanosec` option to mirror Autotools `USE_NSEC`](https://lore.kernel.org/git/20260628081806.GA6789@peff.net/)
**Author**: Jeff King

The discussion around D. Ben Knoble’s `USE_NSEC` Meson parity patch expanded into a **broader re-evaluation** of the knob’s necessity. Jeff King’s testing on modern Linux (ext4, ext2, vfat, CIFS, NTFS, FUSE) demonstrates that nanosecond timestamps are preserved correctly, obsoleting historical "racy Git" concerns. However, Peff also identified **interoperability risks** when mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit), which could trigger stat-dirty re-reads.

Patrick Steinhardt proposed an **incremental path forward**: always compile nanosecond support into Git (when the platform allows it) and expose it via a runtime config knob (e.g., `core.useNsec`), but retain the current build-time default (`USE_NSEC` disabled). This avoids auto-detection challenges while making the feature more accessible. The proposal does not address the default value for the config knob (Peff suggests a conservative default of `false`), but it offers a middle ground between Knoble’s build-time parity patch and Patrick’s earlier skepticism about the knob’s existence.

---

## In brief

**`git cat-file --batch-command`** -- Pablo Sabater’s **14-patch GSoC series** is **technically complete** and ready for merging. The series introduces a `remote-object-info` command for querying object metadata from remotes, with **dynamic format validation** and **680 lines of new tests**. A **philosophical debate** over silent failures versus explicit errors remains unresolved.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **2-line patch** fixing a regression in HTTPS proxy URL validation was **merged into `master`**. The fix ensures HTTPS proxy URLs are correctly accepted and configured.

**Reftable memory leak fix** -- Jeff King’s **4-line patch** fixing a memory leak in `reftable_writer_new` was **merged into `pu`**. The fix moves the allocation after input validation, plugging a leak introduced in February 2025.

**French translation update** -- Jiang Xin’s **l10n pull request** for Git 2.55.0 updates `.po` files for 11 languages and includes a mass typo-fix pass for French. The pull request is **ready for merging**.

---

## On the radar

**`git replay --linearize` regression** -- Toon Claes must send a follow-up patch to fix the silent commit dropping issue. The regression is **urgent** and must be addressed before the next release.

**`git repack --geometric --cruft`** -- Taylor Blau’s **10-patch RFC** faces a **correctness issue** in its reachability filtering. The series aims to combine `--geometric` and `--cruft` repack modes but may retain unreachable tags and objects. A fix is needed before merging.

**`git history squash`** -- Harald Nordgren’s **v6 series** is ready for Junio’s final review. The only remaining debate is whether `--reedit-message` (or `--edit`) should be the default.

**ODB abstraction** -- Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use a `source` field awaits substantive review. The series is foundational for pluggable ODB backends.

**`USE_NSEC` runtime configuration** -- Patrick Steinhardt’s proposal to make `USE_NSEC` runtime-configurable (via `core.useNsec`) is gaining traction. The proposal avoids auto-detection challenges but leaves the default value unresolved.