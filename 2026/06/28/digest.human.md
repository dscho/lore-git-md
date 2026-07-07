Here’s the daily digest for **2026/06/28**, covering the Git mailing list’s key developments in a concise, developer-focused overview:

---

### **The day in brief**
A **moderately busy Sunday** (45 emails, 17 threads) with **two critical regressions surfacing**, **performance optimizations landing**, and **security hardening patches finalized**. The standout threads: **`git replay --linearize`’s post-merge regression** (urgent fix needed), **Tian Yuchen’s `paint_down_to_common()` optimization** (100-1000x speedups merged), and **Patrick Steinhardt’s reftable security series** (fuzzing infrastructure now live). L10n updates and CI fixes rounded out the day.

---

### **Notable threads**

#### **`git replay --linearize` regression: commits silently dropped**
**Thread**: *[PATCH 0/3] replay: introduce --linearize option*
**Author**: Toon Claes (v6 series)
**Status**: **Urgent regression** reported by Johannes Schindelin.
**Issue**: When replaying a single branch containing merge commits (e.g., `master~2..master --linearize --onto master~2`), the command **drops intermediate commits**, replaying only the tip. The bug stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which tracked the correct parent for linearization.
**Impact**: Breaks the all-or-nothing flattening guarantee of `--linearize`.
**Next steps**: Toon Claes is expected to send a follow-up patch to restore `replayed_base` or redesign the multi-branch handling logic. **Must be fixed before 2.55.0**.

---

#### **`paint_down_to_common()` optimization merged (100-1000x speedups)**
**Thread**: *[PATCH v4 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories*
**Author**: Tian Yuchen (v4 series)
**Status**: **Fully approved and merged** after fixing a regression in v3.
**Key changes**:
- **Early termination**: The walk halts when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts).
- **Instrumentation**: Trace2 metrics for "commits walked" and hyperfine benchmarks (e.g., `merge-base --all` in a 2.6M-commit monorepo drops from **3.67s → 5ms**).
- **Documentation**: New technical doc (`paint-down-to-common.adoc`) explains the algorithm and termination conditions.
**Why it matters**: A **major performance win** for `git merge-base` in asymmetric histories, with no known regressions.

---

#### **Reftable security hardening: fuzzing infrastructure live**
**Thread**: *[PATCH 0/11] reftable: fix various bugs found by fuzzing*
**Author**: Patrick Steinhardt
**Status**: **Merged into `next`**.
**Key fixes**:
- **OOB reads/writes** in log block reconstruction (patch 5/11).
- **NULL pointer dereferences** on truncated tables (patch 11/11).
- **`abort()` calls** in the parser.
**Infrastructure**: Added a **libFuzzer-based fuzzer** (runs for 2+ hours in CI without new findings).
**Why it matters**: Critical for the reftable backend’s stability; the fuzzer sets a new bar for security testing in Git.

---

#### **`git cat-file --batch-command` ready for merging**
**Thread**: *[GSoC] [PATCH v14 00/13] cat-file: add --batch-command remote-object-info*
**Author**: Pablo Sabater
**Status**: **Technically complete** (v15 addresses all feedback).
**Key features**:
- **`remote-object-info` command**: Queries metadata (e.g., object size) for up to 10,000 objects in a single command.
- **Dynamic format validation**: Placeholders (e.g., `%(objectsize)`) are validated against server-advertised capabilities.
- **Security**: Strict protocol v2 enforcement and input validation.
**Open question**: Should the client **fail explicitly** or **continue silently** when metadata is missing? No consensus yet, but the behavior is documented.
**Why it matters**: A **significant step forward** for partial clone workflows.

---

#### **CI hangs in `t5551`/`t5559`: Apache timeout fix merged**
**Thread**: *macOS CI hang in t5551/t5559 – root cause and fix*
**Author**: Jeff King (Peff)
**Status**: **Merged** (3-patch series).
**Fix**: Increased Apache’s `Timeout` directive from 300s → 600s in `t/lib-httpd/apache.conf`.
**Optimizations**:
- Isolated the "many-tags" test case into a dedicated repository.
- Packed refs after creating 100,000 tags (reduces `ls-refs` time from **1.2s → 24ms**).
**Why it matters**: Resolves persistent CI flakes without client-side workarounds.

---

### **In brief**
- **`git history squash` (v6)**: Harald Nordgren’s series is **feature-complete**, with stricter input validation and support for multiple revision arguments (e.g., `@~3.. ^topic`). Ready for Junio’s final review.
- **`excludes_file` libification**: Tian Yuchen’s **2-patch series** (moving the global variable into `struct repo_config_values`) is **ready for `next`** after resolving a guardrail debate.
- **`USE_NSEC` Meson parity**: D. Ben Knoble’s patch adding a `nanosec` option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. Discussion now centers on flipping the default to `true`.
- **French translation update**: Jean-Noël Avila’s pull request for Git 2.55.0 includes a **mass typo-fix pass** alongside string updates. Covers 11 languages.
- **`gitk`/`git-gui` quiet builds**: Harald Nordgren’s patches align translation catalog generation with core Git’s quiet build conventions. **Merged (gitk) or superseded (git-gui)**.

---

### **On the radar**
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for a **reachability filtering flaw** that could misplace reachable objects in cruft packs.
- **`git branch --delete-merged`**: Harald Nordgren’s **18-iteration series** is feature-complete and ready for Junio’s final assessment.
- **ODB abstraction**: Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` awaits substantive review.

---

### **Editor’s note**
Today’s traffic underscored **two critical regressions** (`git replay --linearize` and `git repack --geometric --cruft`) that demand urgent attention. The **`paint_down_to_common()` optimization** and **reftable security hardening** are bright spots, demonstrating Git’s ability to deliver **major performance and security wins** through focused refactoring. The **`USE_NSEC` debate** highlights a recurring tension: balancing backward compatibility with modern filesystem realities. Expect follow-ups on the regressions and ODB abstraction in the coming days.