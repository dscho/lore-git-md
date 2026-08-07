Here’s the digest for **2026/08/06**, structured to highlight the day’s most consequential developments while respecting the absolute rules about merge status and terminology.

---

### The day in brief
**2026/08/06** was a **heavy-traffic Thursday** (118 emails, 32 active threads) dominated by **integration-ready features**, **performance optimizations**, and **procedural wrap-ups**. The standout events:
- **`git branch --delete-merged`** (Harald Nordgren) received final maintainer approval and is now **queued in `next`**.
- **`git history squash`** (Harald Nordgren) remains **technically complete** but stalled on **procedural friction**—Junio’s diagnosis of “bottomless time sink” review cycles may force a maintainer takeover.
- **`paint_down_to_common()` optimization** (Tian Yuchen & Kristofer Karlsson) is **queued in Junio’s tree** and awaiting additional reviewers.
- **`git repack --drop-filtered`** (Siddharth Shrimali) is **feature-complete** and needs only **mechanical test-script fixes** before merging.
- **ODB abstraction** saw two series land: **on-disk structure pluggability** (Patrick Steinhardt) is **approved and queued**, while **packfile writes in `receive-pack`** (Justin Tobler) is under active review.

---

### Notable threads

#### 1. `git branch --delete-merged` graduates to `next`
**Headline**: `--delete-merged` lands after 25 iterations
**Author**: Harald Nordgren
**Status**: **Merged to `next`** (Junio’s final sign-off in [2026/08/06/17-36-00](2026/08/06/17-36-00))

The 25th iteration of Harald Nordgren’s `--delete-merged` series—providing safe, automated local-branch cleanup with **order-independent stacked-branch protection**, per-branch opt-out, and `--dry-run` preview—has been **fully approved** and **queued in `next`**. Junio’s final review confirmed the v25 changes (wrapped test command lines) were purely mechanical, and Phillip Wood’s earlier technical concerns (push-detection logic, stacked-branch protection) were resolved in v24. The series is now **implementation-complete**, with only a follow-up planned to extend the stacked-branch protection to `git branch -d` for consistency.

**Key files**: `builtin/branch.c`, `t/t3200-branch.sh`
**Safety net**: `branch.<name>.deleteMerged=false` config key, `--dry-run` preview, and explicit rejection of checked-out branches.

---

#### 2. `git history squash` stalls on procedural friction
**Headline**: “Bottomless time sink” review cycles risk maintainer takeover
**Author**: Harald Nordgren
**Status**: **Technically complete (v12)** but **procedurally stalled**

Harald Nordgren’s **12-patch `git history squash`** series—collapsing commit ranges into a single commit with autosquash-style message editing—remains **feature-complete** and **ready for integration**, but Junio’s latest email ([2026/08/06/09-19-27](2026/08/06/09-19-27)) diagnoses the root cause of the stall: **rapid rerolls invalidated nearly-complete reviews**, creating a “bottomless time sink” that discouraged participation. Harald has signaled **potential disengagement**, leaving the series’ future contingent on whether he can **coordinate with reviewers to avoid invalidating their work**. If not, a maintainer takeover may be necessary to advance the topic.

**Key files**: `builtin/history.c`, `t/t3455-history-squash.sh`
**Open question**: Will Harald adapt to the project’s review rhythm, or will the series require a new maintainer?

---

#### 3. `paint_down_to_common()` optimization queued
**Headline**: 100–1000× speedup for asymmetric merge-base queries
**Authors**: Tian Yuchen & Kristofer Karlsson
**Status**: **Queued in Junio’s tree** (v7); awaiting additional reviewers

The **9-patch series** optimizing `paint_down_to_common()`—terminating early when one side of a merge-base query exhausts its commit queue—has been **queued** after addressing all review feedback. The series introduces **Trace2 instrumentation** to report step counts, deterministic test assertions for all commit-graph modes, and a **topological ceiling** for v1 commit-graphs. Benchmarks show **100–1000× speedups** for asymmetric queries (e.g., repositories with import grafts or shallow histories). Junio’s latest email ([2026/08/06/19-06-39](2026/08/06/19-06-39)) confirms the series is **procedurally unblocked** but notes that **review bandwidth remains the bottleneck**.

**Key files**: `commit-reach.c`, `t/t6600-test-reach.sh`
**Performance impact**: Step counts drop from 81 to 9 in `merge-base --all commit-9-9 commit-9-1`.

---

#### 4. `git repack --drop-filtered` nears merge
**Headline**: Partial clones reclaim disk space safely
**Author**: Siddharth Shrimali
**Status**: **Feature-complete (v3)**; needs mechanical test-script fixes

Siddharth Shrimali’s **7-patch series** adding `--drop-filtered` to `git repack`—safely removing locally cached promisor blobs exceeding a size threshold—is **feature-complete** and addresses all prior feedback. Junio’s latest review ([2026/08/06/22-19-08](2026/08/06/22-19-08)) identified **three mechanical issues**:
1. **SYNOPSIS mismatch** in `Documentation/git-repack.adoc` (extra closing bracket).
2. **Missing newline** in the usage string.
3. **Test script style** (bare `grep` instead of `test_grep`).

These are **trivial fixes** that do not affect functionality. The series is otherwise **ready for `next`**, with the safety guards (merge/rebase/cherry-pick checks, index validation) reframed as UX optimizations.

**Key files**: `builtin/repack.c`, `t/t7706-repack-drop-filtered.sh`
**Safety**: Only promisor objects are targeted; locally created objects are untouched.

---

#### 5. ODB abstraction: two series land
**Headline**: On-disk structure pluggability approved; packfile writes in review
**Authors**: Patrick Steinhardt (approved), Justin Tobler (in review)
**Status**:
- **`ps/odb-make-creation-pluggable`**: **Approved and queued** (v4).
- **`jt/odb-transaction-write-pack`**: **Under active review** (6 patches).

Patrick Steinhardt’s **6-patch series** decoupling on-disk ODB structure creation from repository setup has been **fully approved** after adopting Toon Claes’s alternative design (explicit `odb_new()` calls at every call site). The series is now **queued in `seen`** and expected to graduate to `next` shortly.

Justin Tobler’s **6-patch series** extending the ODB transaction system to support pluggable packfile writes in `git receive-pack` is under active review. The series introduces `odb_transaction_write_pack()` and refactors `receive-pack` to eliminate global state. The final patch ([2026/08/06/21-38-59](2026/08/06/21-38-59)) switches `receive-pack` to the new interface, enabling backend-agnostic packfile processing.

### Key files

- **Pluggable creation**: `odb.c`, `setup.c`
- **Packfile writes**: `builtin/receive-pack.c`, `odb/transaction.c`

---

### In brief
- **`git bisect --reset-when-found`**: **Approved and queued in `next`** (Junio’s final sign-off in [2026/08/06/17-36-00](2026/08/06/17-36-00)).
- **`git rebase --update-refs` symref bugfix**: **Design resolved** (Junio concedes to Phillip Wood’s robustness approach in [2026/08/06/20-12-11](2026/08/06/20-12-11)).
- **Trailer parsing bugfix**: **Ready for integration** (Kristoffer Haugsbakk’s `://` exclusion logic approved; Peff’s `starts_with()` readability tweak pending).
- **`git cat-file --batch-command` `%(objecttype)`**: **Needs review** (Pablo Sabater’s v4 series; Peff’s transport-layer refactoring proposed in [2026/08/06/17-17-14](2026/08/06/17-17-14)).
- **`git-refs(1)` documentation**: **Queued in `next`** (Kristoffer Haugsbakk’s warning admonition and cross-reference approved in [2026/08/06/17-32-19](2026/08/06/17-32-19)).
- **`gitk` color-preference dialog**: **Under review** (Johannes Sixt’s 5-patch UI overhaul in [2026/08/06/06-40-13](2026/08/06/06-40-13)).
- **`git history` completion**: **Under review** (Vincent Mailhol’s 4-patch v2 series in [2026/08/06/20-27-35](2026/08/06/20-27-35)).

---

### On the radar
- **`git whatchanged` deprecation**: Junio’s procedural nudge ([2026/08/06/18-09-37](2026/08/06/18-09-37)) reminds users to find replacements before reporting continued use.
- **`git rebase` commit-graph regression**: Philippe Blain reports a fatal error when submodule pointers change ([2026/08/06/19-44-25](2026/08/06/19-44-25)); bisected to `bb5da75d61`.
- **`git fetch` feature request**: Douglas Puchalski proposes decoupling default fetch refspecs from explicit branch mappings ([2026/08/06/22-26-05](2026/08/06/22-26-05)).

---

### Editorial note
The day’s traffic underscored **two recurring themes**:
1. **Review bandwidth as a bottleneck**: The `paint_down_to_common()` and `git history squash` threads highlight how **rapid rerolls can discourage participation**, even for technically sound series.
2. **ODB abstraction maturing**: With two series landing (on-disk pluggability, packfile writes), the **pluggable ODB effort is transitioning from design to implementation**, with `receive-pack` as the next major milestone.