Here’s the digest for **2026/07/03**, covering the day’s key developments in Git development:

---

### **The day in brief**
A busy day with **106 emails across 26 threads**, dominated by **feature additions, bugfixes, and infrastructure improvements**. The most urgent items:
- **`git history drop`** (Patrick Steinhardt) is **fully merged** after Junio fixed a whitespace nit.
- **`git replay --linearize`** (Toon Claes) is **technically complete** but awaits resolution of a **symlink-handling inconsistency** with `gitdir`.
- **Memory leaks in non-default hash implementations** (Jeff King) sparked a **design debate** about idempotency in `git_hash_discard()`.
- **`GIT_TEST_LONG` tests** (Patrick Steinhardt) are now **CI-ready** after efficiency fixes.

---

### **Notable threads**

#### **1. `git history drop` subcommand merged**
**Headline**: Patrick Steinhardt’s 11-patch series adding `git history drop` (remove a commit and replay descendants) is **fully merged** after Junio C Hamano fixed a minor whitespace nit in `reset.c`.
**What it does**: Modernizes the reset API, advances `the_repository` removal, and adds conflict detection for bare repositories.
**Why it matters**: Expands Git’s history-editing toolkit alongside `rebase` and `commit --amend`.
**Next steps**: None; the series is closed.

---

#### **2. `git replay --linearize` symlink inconsistency**
**Headline**: Toon Claes’s `git replay --linearize` (flatten merge commits) is **ready for review**, but Patrick Steinhardt identified a **behavioral inconsistency**: `worktree` resolves symlinks to real paths, while `gitdir` matches both symlinked and real paths.
**What’s at stake**: Users may expect consistent behavior between the two conditions.
**Author’s response**: Pending. The fix could either align `worktree` with `gitdir` or document the divergence.
**Next steps**: Resolve the inconsistency before merging.

---

#### **3. Memory leaks in non-default hash implementations**
**Headline**: Jeff King’s 9-patch series plugs leaks in OpenSSL/libgcrypt SHA-256 backends, but reviewers (Patrick Steinhardt, Brian Carlson) **advocate for idempotent `git_hash_discard()`** to simplify error paths.
**Key debate**: Should `git_hash_discard()` be safe to call multiple times (like `strbuf_release()`), or is the current flag-based approach sufficient?
**Author’s stance**: Jeff proposed an `active` flag in `struct git_hash_ctx` to track initialization state, deferring strict idempotency to a follow-up.
**Why it matters**: Affects Rust integration and long-term maintainability.
**Next steps**: Jeff to submit a follow-up patch for idempotency.

---

#### **4. `GIT_TEST_LONG` tests now CI-ready**
**Headline**: Patrick Steinhardt’s 9-patch series makes `GIT_TEST_LONG` tests **reliable and efficient** for CI, fixing hangs and reducing disk usage.
**Key changes**:
- **GitLab CI** now runs `GIT_TEST_LONG` on integration branches (excluding Windows due to RAM limits).
- **macOS RAM-disk workaround removed** (trading 7–12 minute runtime increase for stability).
- **Tests refactored** to use less disk space (e.g., `t5608-clone-2gb.sh` now deletes repos immediately).
**Why it matters**: Ensures expensive tests are visible in CI without consuming excessive resources.
**Next steps**: Junio to queue the series.

---

#### **5. Reftable hardening series**
**Headline**: Patrick Steinhardt’s 12-patch series hardens the reftable backend against corrupted files, **fully reviewed and ready for merging**.
**What it fixes**: Out-of-bounds reads/writes, NULL pointer dereferences, and `abort()` calls triggered by maliciously corrupted reftable files.
**Key addition**: New fuzzing infrastructure (libFuzzer + Meson) to prevent regressions.
**Why it matters**: Critical for the reftable backend’s stability, though exploitation requires local disk access.
**Next steps**: Junio to merge.

---

### **In brief**
- **`git refs` subcommands merged**: Patrick Steinhardt’s `create`, `delete`, `update`, and `rename` subcommands for `git refs` are **ready for `next`**.
- **`git repo info` prefix queries**: GSoC contributor’s patch to add category-based prefix queries (e.g., `git repo info layout`) received **design feedback** from Junio (prefer glob patterns like `layout.*`).
- **`git rm` pathspec behavior**: Clarified that `git rm -n *.json` recurses into subdirectories due to Git’s pathspec rules (not a bug). Documentation may need updates.
- **`git replay --linearize`**: Junio raised a **procedural nit** about authorship attribution but no technical blockers.
- **`git history` signing**: Souma’s patch series teaches `git history` to sign rewritten commits (`fixup`, `reword`, `split`), **awaiting review**.

---

### **On the radar**
- **`USE_NSEC` debate**: Ongoing discussion about whether to **flip the default to `true`** (Brian Carlson) or **deprecate the knob** (Patrick Steinhardt). Knoble confirmed XFS preserves nanosecond timestamps.
- **Worktree diff performance**: Peff’s patch to enable nanosecond-aware timestamp comparisons **awaits validation** of edge cases (kernel inode cache eviction).
- **`paint_down_to_common()` optimization**: Tian Yuchen’s series is **ready for `next`** after a procedural rebase.

---

### **Editorial note**
Today’s traffic reflects Git’s **maturing infrastructure** (reftable hardening, ODB abstraction) and **user-facing improvements** (`git history`, `git refs`). The **memory-leak debate** highlights a tension between pragmatic fixes and long-term API design, while the **`GIT_TEST_LONG` series** shows the project’s commitment to CI reliability. Expect the **`git replay --linearize` symlink issue** and **`USE_NSEC` default** to resurface soon.