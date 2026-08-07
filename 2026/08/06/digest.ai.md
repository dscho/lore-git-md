August 6, 2026

The day in brief
A busy day on the Git mailing list (118 emails, 32 threads) saw **performance optimizations graduate**, **new features land in `next`**, and **long-running series near completion**. The **`paint_down_to_common()` optimization** (100-1000x speedup for asymmetric merge-base queries) is now queued in Junio’s tree, while **`git branch --delete-merged`** and **`git bisect --reset-when-found`** are ready for `next`. A **regression in `git rebase` with submodule symrefs** surfaced, and **Git for Windows build adjustments** continue to upstream. The **ODB abstraction effort** advanced with new transaction interfaces, and **`git history squash`** remains stalled on procedural friction. **Documentation and completion** patches dominated the "In brief" section, with **`git-refs(1)`** and **`git history`** completion landing.

---

### Notable threads

**`paint_down_to_common()` optimization graduates**
The nine-patch series by Tian Yuchen and Kristofer Karlsson, optimizing merge-base computation for one-sided histories, is now **queued in Junio’s tree** after addressing all review feedback. The final patch removes the commit-date fallback entirely, relying on the new side-exhaustion optimization to deliver **100-1000x speedups** in asymmetric queries (e.g., repositories with import grafts). The series is a clear win for performance, with no regressions on smaller repositories. **Action**: Expect graduation to `master` in the next cycle.

**`git branch --delete-merged` lands in `next`**
Harald Nordgren’s **v25 iteration** of the `--delete-merged` feature is **ready for integration testing**, with Junio’s final sign-off. The series provides safe automated branch cleanup with **order-independent stacked-branch protection**, per-branch opt-out, and `--dry-run` preview. All blocking issues (push-detection logic, stacked-branch protection) were resolved in v24, and the v25 changes are purely mechanical (test command line wrapping). **Action**: Junio will queue the series in `next` shortly.

**`git bisect --reset-when-found` ready for `next`**
Junio explicitly approved Harald Nordgren’s `--reset-when-found` feature for `git bisect`, calling the v6 series "pretty happy with where we are." The patch adds an auto-reset option after finding the culprit, eliminating the need for an explicit `git bisect reset` in automated workflows. The implementation is complete, with all feedback addressed. **Action**: Expect graduation to `next` in the next "What's cooking" cycle.

**Regression: `git rebase` with submodule symrefs**
Philippe Blain reported a **regression in `git rebase`** when commit-graph is enabled, triggered by submodule pointer changes. The error (`fatal: invalid commit position. commit-graph is likely corrupt`) bisects to Patrick Steinhardt’s `bb5da75d61` ("commit: use commit graph in `lookup_commit_reference_gently()`"). The issue is **specific to submodules** and disappears when commit-graph is disabled. **Action**: Patrick Steinhardt and the commit-graph maintainers will need to investigate; a minimal reproducer is pending.

**Git for Windows build adjustments**
Johannes Schindelin’s 12-patch series upstreaming Git for Windows build improvements continues to progress. Junio’s review of patch 12/12 identified a **cross-platform inconsistency** in `MINGW_PREFIX` handling between Make and Meson, which Dscho has committed to fixing. The series addresses hard-coded assumptions, deprecated build artifacts, and a locale-handling regression in the MSYS2 runtime. **Action**: A v2 reroll is expected to address the build-system mismatch.

**ODB abstraction: `odb_transaction_write_pack()`**
Justin Tobler’s six-patch series extends the ODB transaction system to support **pluggable packfile writes in `git receive-pack`**. The final patch introduces `odb_transaction_write_pack()`, completing the switch from hardcoded `git index-pack`/`git unpack-objects` to a backend-agnostic interface. The series is part of the ongoing ODB abstraction effort led by Patrick Steinhardt. **Action**: Reviewers should focus on the transaction lifecycle design (e.g., `release` vs. post-commit cleanup) and error handling.

**`git history squash` stalled on procedural friction**
Harald Nordgren’s **v12 iteration** of the `git history squash` series remains stalled due to **procedural friction** with the review process. Junio’s latest email diagnoses the root cause: rapid rerolls invalidated nearly-complete reviews, creating a "bottomless time sink" that discouraged participation. The series is technically complete, but Harald’s engagement is now contingent on adapting to the project’s review rhythm. **Action**: If Harald does not proceed with v13 or coordinate with reviewers, the series may require a new maintainer to advance.

**Documentation: `git-refs(1)` improvements graduate**
Kristoffer Haugsbakk’s two-patch series improving the `git-refs(1)` man page is **queued for `next`**. The patches elevate a ref migration warning into a `WARNING` admonition block and add a cross-reference to `git-maintenance(1)`. Junio’s final sign-off confirms the series is ready for integration. **Action**: Expect graduation to `master` in the next cycle.

---

### In brief

**Performance**
- **`paint_down_to_common()` optimization** (Tian Yuchen, Kristofer Karlsson): Queued in Junio’s tree; 100-1000x speedup for asymmetric merge-base queries.
- **`git repack --drop-filtered`** (Siddharth Shrimali): v3 addresses all feedback; Junio’s review flags test script style violations (fixed in v4).

**Features**
- **`git branch --delete-merged`** (Harald Nordgren): v25 ready for `next`; order-independent stacked-branch protection, per-branch opt-out, `--dry-run`.
- **`git bisect --reset-when-found`** (Harald Nordgren): Junio’s final sign-off; auto-reset after finding the culprit.
- **`git history