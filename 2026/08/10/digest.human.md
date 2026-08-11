Here’s the digest for **2026/08/10**, covering the day’s most consequential developments in Git’s mailing list traffic:

---

### **The day in brief**
A busy Monday (81 emails, 25 threads) with **two high-priority regressions** surfacing—one in `git rebase` with submodules and commit graphs, another in `git maintenance`’s geometric repacking—and a **security flaw** in `gitk`’s custom commands. The **ODB abstraction effort** advanced with new patches, while **GSoC projects** and **usability fixes** rounded out the day. The tone was collaborative but urgent where regressions were involved.

---

### **Notable threads**

#### **1. `git rebase` regression with submodules and commit graphs**
**Headline:** A regression in `git rebase` (bisected to `bb5da75d61`) causes fatal errors (`invalid commit position. commit-graph is likely corrupt`) when submodules are involved. The issue stems from `lookup_commit_reference_gently()` resolving submodule OIDs against the main repository’s commit graph.
**Status:** Under investigation. Patrick Steinhardt asked for confirmation whether the issue persists on `master` (where recent commit-graph fixes may already address it) and a minimal reproducer. **Workaround:** Disable commit graphs (`core.commitGraph=false`).
**Why it matters:** This affects users with submodules, a common workflow in large repositories. The regression was introduced between Git v2.53 and v2.54, making it a blocker for the next release.

---

#### **2. `git maintenance` geometric repacking: concurrency failures and auto-condition bug**
**Headline:** Stefan Haller reported **two critical issues** in `git maintenance`’s geometric repacking:
1. **Concurrency failures:** `prune_packed_objects()` removes fanout directories while concurrent writers attempt to create temporary files, causing "invalid object" errors.
2. **Auto-condition bug:** `geometric-repack.auto` triggers 256× more aggressively than documented, causing excessive background repacks in small repositories.
**Status:** Patrick Steinhardt proposed raising the default `geometric-repack.auto` threshold from 100 to 6700 (matching `gc.auto`) to mitigate the issue, but Stefan declined to test it, leaving the fix unvalidated. The underlying race condition remains unaddressed.
**Why it matters:** These bugs risk data corruption during routine operations and degrade performance. The geometric repacking feature was introduced in Git 2.54, so this is a **regression with real-world impact**.

---

#### **3. `gitk` custom commands: security flaw in shell substitution**
**Headline:** A **critical security flaw** in Tim Wiederhake’s `gitk` patch (adding user-configurable custom commands) allows arbitrary command injection via unquoted placeholder substitution (e.g., `%t` for commit titles). Junio C Hamano provided a concrete exploit example (`title?'; echo no'`).
**Status:** The author is evaluating two fixes:
- Revert to direct command execution (requiring wrapper scripts for advanced use cases).
- Implement proper escaping for all interpolated values.
**Why it matters:** This is a **blocker** for the feature, which aims to extend `gitk`’s workflow flexibility. The discussion highlights the tension between usability and security in Git’s UI tooling.

---

#### **4. ODB abstraction: pluggable packfile generation and transaction fixes**
**Headline:** Justin Tobler’s ODB transaction series (making `git receive-pack` backend-agnostic) saw **critical fixes** for a lockfile lifecycle flaw in the "files" backend, where `.keep` lockfiles were moved during commit but tempfile structures still pointed to pre-migration paths. The fix ensures lockfile paths remain valid post-commit.
**Status:** The series is cooking in `seen`, with follow-ups planned to address Junio’s feedback on transaction lifecycle and source-tracking design debt.
**Why it matters:** This is a key step in Patrick Steinhardt’s **ODB abstraction effort**, which aims to support alternative backends (e.g., reftable, cloud storage). The lockfile flaw could have left `.keep` files stranded, undermining the transaction system’s robustness.

---

#### **5. `git repack --drop-filtered` for partial clones (v4)**
**Headline:** Siddharth Shrimali posted v4 of the `git repack --drop-filtered` series, which allows users to reclaim disk space in partial clones by safely removing locally cached promisor blobs exceeding a size threshold.
**Status:** The series is complete (7/7 patches) and addresses all prior feedback, including Junio’s mechanical fixes. The **safety guards** (merge/rebase checks, index validation) are now framed as UX optimizations, not correctness measures. Junio’s proposed time-based heuristic (avoiding recently fetched objects) remains deferred.
**Why it matters:** This is a **long-awaited feature** for partial clone users, enabling disk-space management without breaking lazy-fetching. The series is production-ready but may see minor tweaks based on reviewer feedback.

---

### **In brief**
- **`git send-email`:** Harald Nordgren’s bugfix for missing "Subject:" headers was approved and marked for `next`.
- **`git worktree add`:** Yoichi NAKAYAMA’s v3 patch improving error messages for ambiguous remote branches addressed Junio’s feedback on commit-message clarity.
- **`git maintenance` promisor packs:** Taylor Blau’s two-patch bugfix for geometric repacking was reviewed by Patrick Steinhardt, who noted a potential edge case (now resolved).
- **`git history` Bash completion:** Vincent Mailhol’s v2 series added completion for the new `git history` command, with D. Ben Knoble advocating for a simplified approach to avoid `--option <value>` syntax.
- **GSoC updates:** Siddharth Shrimali’s Week 11 progress on partial clone disk-space recovery and Siddharth Shrimali’s (separate contributor) `gc-promisor` maintenance task were shared.
- **CI/build system:** Jeff King’s patch to bump the Ubuntu image for static-analysis jobs was contested after SZEDER Gábor’s benchmarks showed Coccinelle 1.3.1 as **4.5× slower** than 1.1.1.

---

### **On the radar**
- **`core.useNanosec`:** D. Ben Knoble’s series converting `USE_NSEC` to a runtime config option needs a decision on where to cache the value (`repo_config_values` vs. `repo_settings`).
- **`uploadpack.lazyFetchTrusted`:** Christian Couder’s series (replacing `GIT_NO_LAZY_FETCH=fromAccepted`) is stalled on a submission format issue but addresses a key security objection.
- **`git rebase` regression:** Florian Schmidt’s report awaits confirmation on `master` and a minimal reproducer.

---

### **Editorial note**
Today’s traffic underscored **three themes**:
1. **Regressions demand urgency:** The `git maintenance` and `git rebase` bugs surfaced with real-world impact, requiring immediate attention.
2. **Security vs. usability:** The `gitk` flaw and `uploadpack.lazyFetchTrusted` debate highlight Git’s balancing act between flexibility and safety.
3. **ODB abstraction matures:** Justin Tobler’s transaction fixes and Patrick Steinhardt’s pluggable packfile generation patches show steady progress toward backend-agnostic Git.