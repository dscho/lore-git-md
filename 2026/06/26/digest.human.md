Here’s the daily digest for **June 26, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A busy day with **86 emails across 17 threads**, dominated by **security hardening, performance optimizations, and refactoring**—plus a late-breaking RFC to combine `git repack --geometric` and `--cruft`. The most urgent items: a **regression in the `paint_down_to_common()` optimization** (now blocked on a fix) and **Junio’s approval of the promisor object refactoring**, which is ready to merge. The `git replay --linearize` series also reached v5, addressing all prior feedback.

---

### **Notable threads**

#### **1. `git cat-file --batch-command` remote object metadata (GSoC v14)**
**Headline**: Final nits addressed; series ready to merge.
The GSoC series implementing `git cat-file --batch-command` for remote object metadata queries (e.g., size) is now **technically complete**. Today’s traffic focused on **minor documentation and style fixes** (e.g., `strtoul_szt()` return type clarification, function naming consistency). Pablo Sabater addressed Junio’s feedback on patch 2/13, switching to `uintmax_t` for cross-platform safety. The series is **security-hardened**, with dynamic format placeholder validation and memory safety fixes, and has **no unresolved objections**. Expect this to land in `next` shortly.
*Key files*: `builtin/cat-file.c`, `fetch-object-info.c`, `Documentation/git-cat-file.adoc`.

---

#### **2. `git replay --linearize` (v5)**
**Headline**: Boolean refactoring reverted; series now addresses all feedback.
Toon Claes **released v5** of the `--linearize` option for `git replay`, reverting the controversial `enum replay_mode` → boolean refactoring in favor of **detailed code comments** to clarify base-commit selection. The new version also **fixes a bug** in `--onto` handling for divergent branches and adds a test case. Junio’s feedback on patch 1/3 (style nit) was addressed, and the **interface design debate** (whether to mirror `git rebase`’s three-mode design) remains open but doesn’t block merging. The series is **ready for Junio’s final review**.
*Key files*: `replay.c`, `t/t3650-replay-basics.sh`.

---

#### **3. Reftable security hardening (v1)**
**Headline**: Fuzzing infrastructure added; fixes for OOB reads/writes.
Patrick Steinhardt’s **11-patch series** hardens the reftable backend against corrupted files, fixing **out-of-bounds reads/writes, NULL pointer dereferences, and `abort()` calls** discovered via libFuzzer. Today’s traffic included **Christian Couder’s test hygiene suggestion** (extract a helper for duplicated test code) and **no substantive objections**. The series adds **Meson build support for fuzzing** and CI integration, making it the first Git subsystem with coverage-guided fuzzing. **Ready for review**.
*Key files*: `reftable/block.c`, `oss-fuzz/fuzz-reftable.c`, `t/unit-tests/`.

---

#### **4. `paint_down_to_common()` optimization (v3)**
**Headline**: Regression blocks series; test suite catches bug.
The **post-merge follow-up** to Tian Yuchen’s merge-base optimization hit a snag: **patch 7/8 introduced a regression** by widening a BUG assertion unconditionally, breaking `get_merge_bases_many()` when `min_generation` is unset. Junio ejected the series from `seen`, and Kristofer Karlsson confirmed the test suite caught the bug. A **v4 rework** is planned to either revert the problematic change or unify halt conditions while preserving correctness. The rest of the series (patches 1–6/8 and 8/8) remains **stable and ready to merge**.
*Key files*: `commit-reach.c`, `t/t6600-test-reach.sh`.

---

#### **5. Promisor object connectivity checks (v3)**
**Headline**: Junio and Christian Couder approve; ready for `next`.
Patrick Steinhardt’s **5-patch series** refactors connectivity checks to use generic ODB iteration instead of packfile internals, enabling promisor object support for non-packfile backends. **Junio and Christian Couder approved the final patch**, which adds a test verifying the optimization and fallback behavior. The series is **critical for ODB abstraction** and has no unresolved feedback.
*Key files*: `connected.c`, `t/t5616-partial-clone.sh`.

---

#### **6. `git repack --geometric` + `--cruft` (RFC)**
**Headline**: Late-breaking RFC proposes combining repack modes.
Taylor Blau **submitted a 10-patch RFC** to combine `--geometric` and `--cruft` repack modes, enabling repositories to maintain geometric progression of reachable packs while collecting unreachable objects into cruft packs. The series introduces `--stdin-packs=follow-reachable` to `git pack-objects` and wires it into `git repack`, with **8 new tests** covering edge cases. Junio flagged a **potential correctness issue** in the reachability traversal logic (unreachable tags could be incorrectly included). Given the **complexity and timing** (submitted during -rc phase), this may require multiple iterations.
*Key files*: `builtin/pack-objects.c`, `t/t7704-repack-cruft.sh`.

---

### **In brief**
- **`libgit.a` reorganization**: Patrick Steinhardt’s RFC to move sources into `lib/` gained a **Windows CI fix** from Johannes Schindelin. Junio queued it under `ps/libgit-in-subdir`.
- **Ref backend lazy-loading**: Patrick’s **11-patch series** (fixing `includeif.onbranch` recursion) was **merged into `next`** as `ps/refs-onbranch-fixes`.
- **`git history squash`**: Phillip Wood’s feedback on edge cases (e.g., single-commit ranges) prompted a **v6 iteration** to tighten input validation.
- **macOS CI hangs**: The root cause was identified as an **Apache `mod_http2` bug** (upstream 70131). Consensus emerged to **increase Apache’s `Timeout`** as a mitigation.
- **`excludes_file` libification**: Tian Yuchen’s series to move the global variable into `struct repo_config_values` hit a **build-breaking issue** (unused parameter) and a **design debate** over submodule guards. Junio suggested using `UNUSED` as a temporary fix.
- **Rustification**: Feng Wu fixed a **correctness issue** in `ObjectMap::insert()` for mismatched hash algorithms.

---

### **On the radar**
- **`ps/odb-generalize-prepare`**: Patrick’s ODB refactoring to generalize `reprepare()` needs review (Toon Claes raised questions about downcasting and flag usage).
- **`jt/receive-pack-use-odb-transactions`**: Taylor Blau’s series to make `git-receive-pack` backend-agnostic is waiting for author response.
- **`hn/branch-delete-merged`**: A new feature to delete merged branches needs review.

---