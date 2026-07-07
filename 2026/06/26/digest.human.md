Here’s the daily digest for **June 26, 2026**, covering the Git mailing list’s most significant developments:

---

### **The day in brief**
A busy day on the list (86 emails, 17 threads) saw **two major series land in `next`**, a **critical regression identified** in a performance optimization, and **RFCs for ambitious repack and Rust features** debut. The standout: Patrick Steinhardt’s **reftable hardening** and **reference backend fixes** graduated, while Taylor Blau’s **geometric+cruft repack RFC** sparked early debate. A regression in Tian Yuchen’s **merge-base optimization** blocked its progress, and Junio’s "What’s cooking" report set the stage for Git 2.55-rc2’s deep freeze.

---

### **Notable threads**

#### **Reftable hardening lands in `next`**
**Topic:** `ps/reftable-hardening` (11 patches)
**Author:** Patrick Steinhardt
**Status:** **Merged to `next`** (Authoritative status: [Cooking])
**What happened:** A security-focused series hardened Git’s reftable backend against corrupted files, fixing out-of-bounds reads/writes, NULL pointer dereferences, and `abort()` calls during parsing. The patches also introduced **libFuzzer-based fuzzing infrastructure** (Meson/CI integration) to prevent regressions. Christian Couder’s review was minimal (test hygiene suggestions), but the series was deemed ready for `next` due to its critical safety improvements.
**Why it matters:** Reftable is a key part of Git’s future storage strategy, and this series ensures it’s robust against malicious corruption—even if exploitation requires local disk access. The fuzzing infrastructure may be reused for other subsystems (e.g., packfile parsing).

---

#### **Reference backend fixes graduate**
**Topic:** `ps/refs-onbranch-fixes` (11 patches)
**Author:** Patrick Steinhardt
**Status:** **Merged to `next`** (Authoritative status: [Cooking])
**What happened:** A long-running effort to resolve recursive initialization issues in Git’s reference backend (triggered by `includeif.onbranch` conditions) culminated in a **lazy-loading design**. The series defers write-config parsing until the first write operation, eliminating early config reads that could cause recursion. Jeff King (Peff) and Justin Tobler approved the architecture, and Junio merged it after renaming the topic to better reflect its focus on "onbranch" fixes.
**Why it matters:** This foundational change ensures `git commit` respects `onbranch`-scoped settings (e.g., `core.logAllRefUpdates`) and paves the way for future backend modularity (e.g., ODB-based ref stores).

---

#### **Merge-base optimization hits regression**
**Topic:** `kk/merge-base-exhaustion` (8 patches, v3)
**Author:** Tian Yuchen
**Status:** **Blocked** (regression in patch 7/8)
**What happened:** A performance optimization to terminate merge-base walks early when one side’s commit queue is exhausted (yielding **100-1000x speedups** for asymmetric histories) was derailed by a **critical regression**. Patch 7/8 widened a BUG assertion to fire unconditionally, breaking correctness when `min_generation` is unset. Junio ejected the series from `seen`, and the author acknowledged the issue, planning a v4 fix.
**Why it matters:** The optimization targets large repositories (e.g., with import grafts), but the regression underscores the fragility of generation-number invariants. The test suite’s **deterministic step-count assertions** (added in v3) caught the bug, validating the series’ robust instrumentation.

---

#### **Geometric+cruft repack RFC debuts**
**Topic:** `[RFC PATCH 0/10] repack: combine --geometric and --cruft`
**Author:** Taylor Blau
**Status:** **RFC** (Authoritative status: [New Topics])
**What happened:** Taylor proposed combining `git repack --geometric` and `--cruft` modes, which are currently mutually exclusive. The series introduces `--stdin-packs=follow-reachable` to `git pack-objects`, ensuring only reachable objects from rolled-up packs are included in geometric packs, while unreachable objects are collected into cruft packs. Junio’s early review flagged a **potential correctness issue** in the reachability traversal logic (unreachable tags could be incorrectly retained).
**Why it matters:** This would streamline maintenance workflows for large repositories, but the complexity of the two-phase traversal and the timing (submitted during -rc phase) suggest it may need significant iteration.

---

#### **`git history squash` edges closer**
**Topic:** `hn/history-squash` (4 patches, v5)
**Author:** Harald Nordgren
**Status:** **Needs review** (Authoritative status: [Cooking])
**What happened:** A feature to fold commit ranges into a single commit (`git history squash`) addressed feedback from Phillip Wood and Junio, tightening input validation (rejecting single-commit ranges) and clarifying edge cases (e.g., `fixup!`/`squash!` commits). The series now rejects operations with refs pointing inside the squashed range by default, with advice to use `--update-refs=head`.
**Why it matters:** This avoids surprising users by silently orphaning refs (e.g., bisection markers). The design aligns with `git rebase`’s ref-handling patterns, but Phillip’s questions about merge commits with external parents remain open.

---

### **In brief**
- **`git cat-file --batch-command` security series (v14):** Pablo Sabater addressed Junio’s feedback on `strtoul_szt()` (switching to `uintmax_t` for cross-platform safety). The series is **ready for merging** after 14 iterations, with only commit message nits remaining.
- **`git replay --linearize` (v5):** Toon Claes reverted the controversial `enum replay_mode` refactoring, replacing it with detailed code comments. The series is **technically complete** but awaits Junio’s final review.
- **Libification effort:** Tian Yuchen’s patch to move `excludes_file` into `struct repo_config_values` hit a **build-breaking issue** (unused parameter) and a **design debate** over submodule guards. Junio suggested using `UNUSED` annotations as a temporary fix.
- **Rustification:** A patch from Feng Wu fixed a correctness issue in `ObjectMap::insert()` by validating hash algorithms, part of the ongoing Rust effort.
- **CI hangs:** The macOS CI hang in `t5551`/`t5559` was traced to Apache’s 300-second timeout. The consensus is to **increase the timeout** rather than mask the issue with client-side mitigations.

---

### **On the radar**
- **`jt/receive-pack-use-odb-transactions` (6 patches):** A refactoring to make `git-receive-pack` backend-agnostic by using ODB transactions. Waiting for author response.
- **`ps/odb-drop-whence` (7 patches):** Removes the `whence` field from `struct object_info`, making backend-specific info opt-in. Depends on `ps/odb-source-packed`.
- **`tb/midx-incremental-custom-base` (3 patches):** Fixes reachability closure for bitmaps in incremental MIDX writes. Needs review.

---

### **Editor’s note**
Today’s traffic was dominated by **performance optimizations** (merge-base, repack) and **security hardening** (reftable), with a sprinkle of **libification** and **Rust work**. The regression in Tian’s series is a reminder that even well-tested optimizations can hit subtle invariants, while Taylor’s repack RFC shows the project’s appetite for ambitious refactoring—even during feature freeze. Junio’s "What’s cooking" report confirms that only bugfixes and docs will graduate to `master` until Git 2.55 final, so expect the next few days to focus on stabilization.