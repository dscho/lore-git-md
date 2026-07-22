Here’s the daily digest for **July 21, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A **moderately busy** day with **46 emails across 20 threads**, dominated by **performance regressions, bugfixes, and feature refinements**. Key highlights:
1. **ODB performance regression** in `git receive-pack` (Atlassian) sparked a **design debate** about balancing abstraction vs. real-world constraints.
2. **`git history squash` v10** received final sign-off, clearing the way for integration.
3. **`git cat-file --batch-command` remote-object-info** (v20) addressed last-minute nits, now **technically complete**.
4. **New features** (`git rebase --continue --[no-]edit`, `git repo info` path keys) and **platform-specific fixes** (macOS fsmonitor, submodule URL aliases) rounded out the day.

---

### **Notable threads**

#### **1. ODB Performance Regression: Abstraction vs. Real-World Constraints**
**Topic**: Performance regression in `git receive-pack` connectivity checks after ODB rework (commit `a593373b09`).
**What’s new**: Junio C Hamano reframed the issue as an opportunity to **optimize the ODB abstraction for specialized use cases** (e.g., connectivity checks), proposing:
- **"Fast" vs. "slow" lookups** within backends to enable cross-backend optimizations.
- **Pre-loading loose object names** into a Bloom filter to short-circuit expensive `lstat()` calls.
**Why it matters**: The regression (12% slowdown on fast filesystems; worse on NFS) exposes a tension between the ODB abstraction’s design goals and real-world performance needs. The discussion may shape how Git handles **pluggable backends** in the future.
**Status**: Awaiting Patrick Steinhardt’s (ODB rework author) response on whether the abstraction can accommodate these optimizations.

---

#### **2. `git history squash` v10: Final Sign-Off**
**Topic**: Harald Nordgren’s `git history squash` subcommand (v10).
**What’s new**: **Matt Hunter** (who raised usability concerns in v9) gave the **final sign-off**, confirming all issues (reflog message truncation, `rev-list` option sanitization) are resolved.
**Key features**:
- Folds commit ranges into a single commit while preserving descendant history.
- Handles merge commits, `fixup!`/`squash!`/`amend!` markers, and `--reedit-message` (matching `git rebase -i --autosquash`).
- Rejects operations with interior refs by default (with advice for `--update-refs`).
**Status**: **Ready for integration** (Junio’s "Will replace" sign-off from v7 still stands). The series is a foundational piece for future `git rebase --autosquash` performance improvements.

---

#### **3. `git cat-file --batch-command` remote-object-info: v20 Addresses Nits**
**Topic**: Pablo Sabater’s security-hardened `git cat-file --batch-command remote-object-info` (v20).
**What’s new**: Karthik Nayak’s review focused on **cosmetic nits**:
- **Documentation**: Restored `CAVEATS` references for `objectsize:disk` and `deltabase` atoms (questioned as intentional).
- **Style**: Reformatted `EXPAND_DATA_INIT` macro (attributed to a style-check script).
**Status**: **Technically complete** and queued for `next`. The series enables **efficient remote object metadata queries** (e.g., size) without downloading objects, with dynamic capability negotiation and robust edge-case handling.

---

#### **4. `git rebase --continue --[no-]edit`: Design Debate**
**Topic**: Hugo Sales’ proposal to add `--[no-]edit` to `git rebase --continue`.
**What’s new**: Junio C Hamano **raised substantive concerns**:
- **Necessity**: The existing workaround (`GIT_SEQUENCE_EDITOR=: git rebase --continue`) already achieves the goal.
- **Config scope**: `rebase.noEdit` could silently override user intent during interactive rebase steps (`edit`/`reword`).
- **Edge cases**: Behavior may differ when rebase stops for reasons other than conflicts.
**Status**: Awaiting author’s response. The patch is **under-motivated** in its current form.

---

#### **5. `git repo info` Path Keys: `path.grafts` Design Resolved**
**Topic**: K Jayatheerth’s `git repo info` path keys (e.g., `path.grafts`, `path.index`).
**What’s new**: The **`path.grafts` design debate** is resolved: the command will **return the grafts file path unconditionally** (even if the file doesn’t exist), aligning with `git rev-parse --git-path` precedent.
**Open question**: Should `path.index` return an empty string in bare repositories? (Lucas Seiki Oshiro prefers this to avoid misleading scripts.)
**Status**: Awaiting further review on the remaining patches.

---

### **In brief**
- **`git send-email` completion**: Yury Norov’s patch to include patch files in completion lists sparked a **usability debate**. SZEDER Gábor argued shorter completion lists (offering only refs) are more usable, while Junio preferred filenames. A v2 is expected.
- **Swift userdiff driver**: Shlok Kulshreshtha’s v2 patch (adding Swift support) is **queued in `next`**, pending validation from Swift users.
- **Packfile URI concurrency fix**: Ted Nyman’s v3 series (fixing race conditions in HTTP downloads) addressed Junio’s feedback (splitting the `--index-pack-arg` cleanup into a separate patch). **Ready for review**.
- **`the_repository` removal**: René Scharfe’s lockfile/tempfile API series is **queued in `seen`**, with Patrick Steinhardt deferring a compile-time guard to a future patch.
- **`git show-branch` slab conversion**: Gatla Vishweshwar Reddy’s v10 patch fixed a **division-by-zero crash** (uninitialized `name_slab`). **Ready for Junio’s confirmation**.
- **macOS fsmonitor fix**: Tamir Duberstein’s patch (flushing pending FSEvents before cookie wait) **eliminated timeouts** on loaded systems. **Ready for review**.
- **Submodule URL alias fix**: Éric NICOLAS’s patch fixed a regression where `git submodule update --remote` failed to match remotes using `url.<base>.insteadOf`. **Ready for review**.

---

### **On the radar**
- **`paint_down_to_common()` optimization**: Kristofer Karlsson proposed simplifications (e.g., deferring commit-date fallback cleanup) to ease review. The series is **queued but stalled** due to reviewer availability.
- **`git config --global` includes**: Hendrik Jaeger’s follow-up highlighted the **usability gap** between documented and actual behavior (includes not respected by default). No patch proposed yet.
- **`git stash push` sparse-index optimization**: Ted Nyman’s series (avoiding index expansion for cone-internal pathspecs) is **cooking in `next`** after Junio’s merge signal.

---

### **Editorial notes**
- **Performance vs. abstraction**: The ODB regression thread underscores a recurring theme: **Git’s abstractions must balance purity with real-world constraints** (e.g., NFS deployments). Expect this debate to resurface as the ODB rework matures.
- **Feature readiness**: Two major features (`git history squash`, `git cat-file --batch-command remote-object-info`) cleared their final hurdles, while others (`git rebase --continue --[no-]edit`) face design scrutiny.
- **Platform-specific fixes**: macOS fsmonitor and submodule URL alias patches highlight Git’s **growing reliance on platform-specific optimizations** to maintain performance.