# The Git Project Daily Digest
**2026/06/28 (Sunday) – A Sunday of Regressions, Optimizations, and Final Touches**

The day’s **45 emails across 17 threads** carried a **lighter-than-usual volume** but a **heavier-than-usual technical weight**. The standout developments: a **post-merge regression in `git replay --linearize`** surfaced late in the day, **Tian Yuchen’s `paint_down_to_common()` optimization series received final approval**, and **Harald Nordgren’s `git history squash` series reached feature-complete status**. The day also saw **final readiness for Pablo Sabater’s GSoC `git cat-file --batch-command` series**, a **CI fix for macOS hangs**, and **last-minute l10n updates** for Git 2.55.0.

---

## Notable threads

### `git replay --linearize` regression: silent commit dropping
**Thread: [PATCH 0/3] replay: introduce --linearize option**

Johannes Schindelin **reported a critical regression** in `git replay --linearize` (merged in v5): the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact scenario. Schindelin provided a **reproducible test case** (`master~2..master` with `--linearize --onto master~2`) and explained why the parameter is necessary to ensure all replayed commits are flattened into a single topology.

**Why this matters**: The regression undermines the feature’s core promise—predictable, all-or-nothing flattening—and risks data loss. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled. The fix is **urgent** and must land before the next release.

### Key technical details

- **Files**: `replay.c`, `builtin/replay.c`
- **Root cause**: Removal of `replayed_base` parameter in `pick_regular_commit()`
- **Test case**: `git replay --linearize --onto master~2 master~2..master`
- **Expected behavior**: All commits replayed (`A->X->Y->Z`)
- **Actual behavior**: Only the tip commit replayed (`A->Z`)

---

### `paint_down_to_common()` optimization: final approval
**Thread: [PATCH v4 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories**

Derrick Stolee **gave the final approval** to Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Key improvements:
- **100-1000x speedups** for asymmetric queries (e.g., `merge-base --all` across a 2.6M-commit monorepo drops from 3.67s to 5ms).
- **Regression fix**: Restored the `min_generation` guard to the generation-monotonicity BUG assertion.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).

The series is **merged-ready** and a **major win for `git merge-base` performance** in asymmetric histories. No regressions are known.

---

### `git history squash` reaches feature-complete status
**Thread: [PATCH v6 0/4] history: introduce `squash` subcommand**

Harald Nordgren’s **v6 series** implementing `git history squash` (folding a commit range into its oldest commit while preserving descendant history) is now **feature-complete**. Key improvements since v5:
- **Stricter input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **Support for multiple revision arguments**: Accepts compound ranges like `@~3.. ^topic`.
- **`--reedit-message` template alignment**: Adopts `git rebase -i`’s squash-message template, with `fixup!` messages fully commented out and `squash!`/`amend!` bodies retained.
- **Expanded documentation**: Clarifies the merge limitation and feature overview.

The series is **ready for Junio’s final review**, with no open technical objections. The only remaining debate is whether `--reedit-message` (or `--edit`) should be the default—a **commit hygiene question** raised by Phillip Wood.

### Key technical details

- **Files**: `builtin/history.c`, `sequencer.c`/`sequencer.h`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`
- **New helpers**: `resolve_squash_range()`, `reject_fixupish_oldest()`, `find_interior_ref()`, `build_squash_message()`
- **Behavior**: Rejects operations with interior refs by default, advising `--update-refs=head`.

---

### `git cat-file --batch-command` final readiness
**Thread: [PATCH v15 0/13] cat-file: add --batch-command remote-object-info**

Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries is **technically complete** and ready for merging. The series introduces a new `remote-object-info` command that lets clients request metadata (e.g., object size) for up to 10,000 objects in a single command, reducing network overhead. Key features:
- **Dynamic format validation**: Placeholders are validated based on server capabilities.
- **Security hardening**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.

The series is a **significant step forward for partial clone workflows**, though a **philosophical debate** over silent failures versus explicit errors remains unresolved. Junio’s **final review is pending**.

---

### CI fix for macOS hangs in `t5551`/`t5559`
**Thread: macOS CI hang in t5551/t5559 – root cause and fix**

Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` is **ready for merging**. The series:
1. **Increases Apache’s `Timeout` directive** from 300 to 600 seconds (patch 1/3).
2. **Isolates the expensive "many-tags" test case** into a dedicated repository (patch 2/3).
3. **Packs refs after creating many tags** to reduce `ls-refs` advertisement time (patch 3/3).

The first two patches are **uncontroversial and sufficient** to resolve the CI hangs; the third patch is **optional** and may be dropped. The series is **low-risk** and directly addresses the root cause (Apache bug 70131).

---

## In brief

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

**`git history --reword` fix** -- Junio’s **bugfix patch** addressing a file stream leak in `git history --reword` is **merge-ready**.

**`greplint.pl` regression** -- Junio identified a **critical oversight** in the `greplint.pl` series: the existing `test_grep` implementation already checks for file existence, yet the bug in `t3420-rebase-autostash.sh` persists. The discussion has shifted to **debugging why the safeguard fails** rather than adding new ones.

---

## On the radar

- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **`git history squash`**: Harald Nordgren’s **v6 series** awaits Junio’s final review.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.
- **`--track=fetch` debate**: Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on workflow concerns.