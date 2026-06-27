Here’s the digest for **2026/06/26**, covering the Git mailing list’s daily traffic:

---

### **The day in brief**
A **heavy but focused** day on the Git mailing list, with **86 emails across 17 threads**. The standout themes:
1. **Security and correctness** dominated, with a **14-patch series hardening the reftable backend** against corrupted files (now fully reviewed) and a **critical regression** in the `paint_down_to_common()` optimization (caught by the test suite).
2. **ODB abstraction and libification** advanced, with Patrick Steinhardt’s `ps/odb-source-packed` and related topics queued for `next`, and a **controversial guard pattern** in the `excludes_file` migration sparking debate.
3. **Repack machinery** saw a **10-patch RFC** to combine `--geometric` and `--cruft` modes, though a **correctness issue** in its reachability filtering may delay merging.
4. **Post-merge follow-ups** included a **fix for `git history --reword`** (file leak) and **clarifications** on the `git replay --linearize` edge case (merge commit divergence).

The day’s tone was **collaborative but technical**, with reviewers catching regressions and design flaws early. The **most urgent action item**: Tian Yuchen’s `paint_down_to_common()` regression, which Junio ejected from `seen` and must be fixed before the series can proceed.

---

### **Notable threads**

#### **Reftable security hardening: 11 patches, now fully reviewed**
**Topic**: `[PATCH 0/11] reftable: security hardening against corrupted files`
**Author**: Patrick Steinhardt
**Status**: **Ready for `next`** (all patches reviewed; only a test-code hygiene suggestion remains).
**What’s new**: Christian Couder’s **surface-level review** of patch 5/11 suggested extracting a helper function (`cl_reftable_write_block`) to reduce test duplication. The series fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in the reftable parser, and adds a **libFuzzer-based fuzzer** to prevent regressions. The fixes are **internal-only** (no user-facing changes) and critical for the reftable backend’s stability.
**Why it matters**: Reftable is a key part of Git’s future, and this series ensures it’s robust against maliciously corrupted files. The fuzzing infrastructure is a **long-term win** for security.

---

#### **`paint_down_to_common()` regression: test suite saves the day**
**Topic**: `[PATCH v3 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories`
**Author**: Tian Yuchen
**Status**: **Blocked by regression in patch 7/8**; Junio ejected the series from `seen`.
**What’s new**: Junio, Derrick Stolee, and Kristofer Karlsson **confirmed a critical regression** in patch 7/8, where an **unconditional BUG assertion** breaks correctness when `min_generation` is not set. The test suite caught the bug in `t6600-test-reach.sh` (test 12), validating the series’ edge-case coverage. Kristofer plans a **v4 fix** (either reverting the problematic change or unifying halt conditions while preserving correctness).
**Why it matters**: The optimization yields **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts), but the regression must be fixed before merging. The test suite’s robustness is a **win for Git’s quality assurance**.

---

#### **ODB abstraction: `excludes_file` migration sparks debate**
**Topic**: `[PATCH v2 0/2] Move excludes_file into repo_config_values`
**Author**: Tian Yuchen
**Status**: **Controversial guard pattern**; Junio objects to the `if (repo != the_repository)` guard in the getter/cleanup functions.
**What’s new**: Junio’s **substantive review** argues the guard **masks bugs** by bypassing BUG() assertions, advocating for its removal or replacement with a stricter check (e.g., `repo->initialized`). SZEDER Gábor also **identified a build-breaking issue** (unused `repo` parameter in `repo_excludes_file()`), which Junio proposed fixing with an `UNUSED` annotation. The series is part of the **libification effort** to eliminate global state.
**Why it matters**: The guard pattern is **reused from prior work**, but Junio’s objection raises questions about **debugging hygiene** in the libification effort. The outcome may set a precedent for future patches.

---

#### **Repack RFC: `--geometric` + `--cruft` with a correctness flaw**
**Topic**: `[RFC PATCH 0/10] repack: combine --geometric and --cruft`
**Author**: Taylor Blau
**Status**: **RFC stage**; Junio identified a **correctness issue** in patch 8/10’s reachability filtering.
**What’s new**: Junio’s **substantive review** of patch 8/10 (the `--stdin-packs=follow-reachable` mode) found that the **two-phase traversal** may retain **unreachable tags and objects**, undermining the separation of reachable/unreachable objects. The series aims to let repositories **maintain geometric packs while collecting cruft**, but the flaw could cause reachable objects to be incorrectly placed in cruft packs.
**Why it matters**: This is a **high-impact workflow improvement** for large repositories, but the correctness issue must be resolved before merging. Taylor’s upcoming role change may affect follow-up bandwidth.

---

#### **`git history --reword` fix: file leak and Windows compatibility**
**Topic**: `[PATCH] history: fix file stream leak and inefficient file handling in reword`
**Author**: Junio C Hamano
**Status**: **Merge-ready**; no objections.
**What’s new**: Junio’s **bugfix patch** addresses a **file stream leak** in `git history --reword` that could cause problems on Windows (where open file handles block other processes). The fix consolidates file handling by opening the target file once in write mode, writing the buffer directly, and closing the stream before launching the editor. The issue was introduced in January 2026.
**Why it matters**: A **straightforward correctness fix** with clear motivation. The Windows compatibility concern gives it urgency.

---

### **In brief**
- **`git cat-file --batch-command` remote object info**: Pablo Sabater’s **14-patch series** (v14) is **ready for merging**, with only **commit message nits** remaining. The series adds a **remote-object-info** command to query object metadata (e.g., size) from remotes via protocol v2, with **dynamic format placeholder validation** based on server capabilities.
- **`git replay --linearize`**: Toon Claes’s **v5 series** is **fully merged**, but Junio identified a **post-merge edge case**: merge commit divergence handling differs from `git rebase --no-rebase-merges`, potentially **silently discarding one side of the merge**. A follow-up patch may be needed to document or adjust this behavior.
- **Ref backend lazy-loading**: Patrick Steinhardt’s **11-patch series** (`ps/refs-onbranch-fixes`) is **merged into `next`**, resolving recursive initialization issues caused by `includeif.onbranch`. The series defers write-config parsing until the first write operation, eliminating early config reads that could trigger recursion.
- **Git 2.55.0 l10n cycle**: Jiang Xin’s **translation kickoff** is in its final day (deadline: June 27). Junio reminded translators to submit their work before the release on June 29.
- **`gitk`/`git-gui` quiet builds**: Harald Nordgren’s **2-patch series** to align translation catalog generation with core Git’s quiet build conventions is **merged** (gitk) or **superseded** (git-gui). Johannes Sixt confirmed the changes will land in the next git-gui release cycle.
- **`git history squash`**: Harald Nordgren’s **v6 series** is **imminent**, addressing feedback on **input validation** (rejecting single-commit ranges and non-ancestor ranges) and **edge cases** (handling `fixup!`/`squash!` commits, merge commits with external parents). Junio highlighted a **usability limitation** of the single-argument design, which cannot support complex workflows (e.g., excluding a branch).
- **macOS CI hangs**: Michael Montalbo and Patrick Steinhardt **confirmed Apache’s `mod_http2` bug 70131** as the root cause of CI hangs in `t5551`/`t5559`. The consensus is to **increase Apache’s `Timeout` directive** (e.g., from 300 to 600 seconds) as a pragmatic fix, avoiding client-side mitigations.
- **ODB generalization**: Patrick Steinhardt’s **2-patch series** to generalize the `reprepare()` callback into a `prepare()` callback with a flush flag received **surface-level feedback** from Toon Claes, who questioned **downcasting behavior** and **flag usage consistency** in the `packed` backend.
- **Promisor object connectivity**: Patrick Steinhardt’s **4-patch series** to generalize connectivity checks for promisor objects is **fully reviewed and approved** by Christian Couder and Junio. The series replaces `find_pack_entry_one()` with `odb_for_each_object_ext()`, enabling backend-agnostic connectivity checks.
- **Rustification**: A **standalone patch** to validate hash algorithms in `ObjectMap::insert()` was submitted as part of the Rustification effort. The patch enforces the invariant that both object IDs must use the expected hash algorithms, returning a new `ObjectMapInsertError` for mismatches.

---

### **On the radar**
- **`paint_down_to_common()` regression**: Tian Yuchen must fix the **unconditional BUG assertion** in patch 7/8 before the series can proceed. The fix will likely involve either reverting the problematic change or unifying halt conditions while preserving correctness.
- **`excludes_file` migration**: Tian Yuchen needs to address Junio’s **guard pattern objection** and SZEDER’s **build-breaking issue** (unused `repo` parameter). The outcome may influence future libification work.
- **Repack RFC**: Taylor Blau’s **10-patch series** to combine `--geometric` and `--cruft` modes faces a **correctness issue** in its reachability filtering. The fix may require reworking the two-phase traversal logic.
- **`git history squash`**: Harald Nordgren’s **v6 series** is imminent, with **stricter input validation** and **documentation updates** planned. Junio’s usability concern about the single-argument design remains open.
- **`git replay --linearize` edge case**: Junio’s post-merge observation about **merge commit divergence** may prompt a follow-up patch to document or adjust the behavior.