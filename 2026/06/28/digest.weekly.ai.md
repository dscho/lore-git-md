# The Git Project Weekly Digest
**2026/06/22 -- 2026/06/28**

## The period in brief
This week saw **418 emails across 118 threads**, with traffic **heavier than average** and a **technical focus** on security hardening, performance optimizations, and architectural refactoring. The standout developments: **Patrick Steinhardt’s reftable security series** (11 patches) landed after rigorous fuzzing, **Tian Yuchen’s `paint_down_to_common()` optimization** (8 patches) delivered 100-1000x speedups for asymmetric histories, and **Pablo Sabater’s GSoC `git cat-file --batch-command` series** (14 patches) reached final readiness. A **post-merge regression** in `git replay --linearize` emerged late in the week, requiring urgent follow-up.

---

## Key developments

### Reftable security hardening lands
Patrick Steinhardt’s **11-patch series** hardening Git’s reftable backend against corrupted files was **merged into `next`** after addressing all review feedback. The series fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in the reftable parser, and introduces a **libFuzzer-based fuzzer** to prevent regressions. Key patches:
- **Patch 5/11**: Fixes a heap-buffer-overflow in log block reconstruction.
- **Patch 11/11**: Fixes an OOB read on truncated tables.
- **Patches 1–2**: Add fuzzing infrastructure and CI integration.
The series is **internal-only** (no user-facing changes) and critical for the reftable backend’s stability. The fuzzer now runs for 2+ hours without finding new issues, setting a new bar for security testing in Git.

---

### `paint_down_to_common()` optimization delivers 100-1000x speedups
Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged**. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Key improvements:
- **Regression fix**: Addressed an unconditional BUG assertion in patch 7/8.
- **Code clarity**: Unified halt conditions in the `paint_state` struct.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).
The optimization is a **major win for `git merge-base` performance** in asymmetric histories, with no known regressions.

---

### `git cat-file --batch-command` reaches final readiness
Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries is **technically complete** and ready for merging. The series introduces a new `remote-object-info` command that lets clients request metadata (e.g., object size) for up to 10,000 objects in a single command, reducing network overhead. Key features:
- **Dynamic format validation**: Placeholders are validated based on server capabilities.
- **Security hardening**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.
The series is a **significant step forward for partial clone workflows**, though a **philosophical debate** over silent failures versus explicit errors remains unresolved.

---

### `git replay --linearize` regression requires follow-up
A **post-merge regression** in `git replay --linearize` (merged in v5) was reported by Johannes Schindelin: the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` logic in `pick_regular_commit()`. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled. The regression is **urgent** and must be fixed before the next release.

---

### Ref backend lazy-loading refactor merged
Patrick Steinhardt’s **11-patch series** modernizing Git’s reference backend infrastructure was **merged into `next`**. The series implements lazy-loading to resolve recursive initialization issues caused by `includeif.onbranch` conditions, replacing an earlier workaround with deferred write-config parsing until the first write operation. Key changes:
- **Dynamic write option handling** for the reftable backend.
- **Recursion guard** in `get_main_ref_store()`.
- **Memory leak fixes** in the `chdir_notify` subsystem.
The series is **foundational for ref backend modularity** and has no user-visible behavior changes unless `includeif.onbranch` is used.

---

### `git branch --delete-merged` reaches feature-complete status
Harald Nordgren’s **18-iteration series** adding `git branch --delete-merged` (safe cleanup of merged local branches) is now **feature-complete**. The command aborts with a clear error if a branch is used as an upstream for another branch, preserving the entire dependency chain. Key features:
- **Per-branch opt-out** (`branch.<name>.deleteMerged=false`).
- **`--dry-run` preview** (matches real deletion output).
- **`--forked` filter** (composable with `--merged`/`--no-merged`).
The series is **well-tested** (497-line test suite) and ready for Junio’s final assessment.

---

### ODB abstraction stack advances
Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use a `source` field (replacing the coarse `whence` enum) was **conceptually approved** by Junio. The series enables **multi-source object resolution** and is a foundational step for pluggable ODB backends. Key changes:
- **Patch 1/6**: Threads `struct odb_source_packed *source` through `packed_object_info()`.
- **Patch 6/6**: Adds documentation for `struct object_info` fields.
The series is **ready for substantive review**, with no objections yet.

---

### `git repack --geometric --cruft` RFC hits correctness snag
Taylor Blau’s **10-patch RFC** to combine `--geometric` and `--cruft` repack modes faces a **correctness issue** in its reachability filtering. Junio identified that the two-phase traversal in `--stdin-packs=follow-reachable` may retain **unreachable tags and objects**, undermining the separation of reachable/unreachable objects. The series aims to let repositories **maintain geometric packs while collecting cruft**, but the flaw could cause reachable objects to be incorrectly placed in cruft packs. A fix is needed before merging.

---

## In brief

**`git history squash`** -- Harald Nordgren’s **v6 series** folds commit ranges into a single commit while preserving descendant history. The series is **functionally complete**, with stricter input validation and support for multiple revision arguments (e.g., `@~3.. ^topic`).

**`git log -L` improvements** -- Karthik Nayak’s **7-patch series** extends `git log -L` to support range-scoped `--stat`, `--check`, and `-G` pickaxe operations. The series is **technically complete** and ready for review.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligns translation catalog generation with core Git’s quiet build conventions. The patches are **merged (gitk) or superseded (git-gui)**.

**CI hangs in `t5551`/`t5559`** -- Jeff King’s **3-patch series** increases Apache’s `Timeout` directive to 600 seconds and isolates the expensive “many-tags” test case. The series is **ready for merging**.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

**`git history --reword` fix** -- Junio’s **bugfix patch** addresses a file stream leak in `git history --reword` that could cause problems on Windows. The fix is **merge-ready**.

**`--track=fetch` debate** -- Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on workflow concerns, despite being technically complete. Junio’s reservations about network operations during checkout without user preview persist.

---

## Looking ahead
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue before the next release.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw. The series’ future may depend on Taylor’s availability after a role change.
- **`git history squash`**: Harald Nordgren’s **v6 series** is ready for Junio’s final review.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.
- **`excludes_file` migration**: Tian Yuchen’s series is ready for `next` after resolving the guardrail debate.