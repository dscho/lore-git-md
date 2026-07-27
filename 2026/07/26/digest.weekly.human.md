# The Git Project -- Weekly Digest for 2026/07/20 -- 2026/07/26

## The period in brief

The week of **2026/07/20--2026/07/26** saw **353 emails across 109 threads**, a **heavy but productive** volume for the Git mailing list. Traffic was **eventful and technically dense**, with **two major feature series landing**, **critical performance regressions diagnosed**, and **long-running refactoring efforts reaching key milestones**. The standout developments: **`git branch --delete-merged`** and **`git cat-file --batch-command remote-object-info`** graduated to `next`, while the **ODB alternates performance regression** sparked an architectural redesign. Smaller but impactful fixes—**`git mv` error messages**, **`git rebase --update-refs` symref handling**, and a **`git worktree add` memory-safety bug**—also cleared final hurdles.

---

## Key developments

### **`git branch --delete-merged` lands after 23 iterations**
Harald Nordgren’s **`git branch --delete-merged`** series, in development since May, **reached technical completion and was approved for merging** after its 23rd iteration. The command introduces **safe, automated local-branch cleanup** with comprehensive safety controls: **order-independent stacked-branch protection**, **per-branch opt-out** (`branch.<name>.deleteMerged=false`), **`--dry-run` preview**, and **repeatable upstream selectors**. The v22 revision resolved the last correctness issue (order dependency in stacked-branch protection) via a **single-pass graph traversal**, and Junio C Hamano’s final approval confirmed the series is "pretty much done." The only agreed follow-up—extending stacked-branch protection to `git branch -d`—will be implemented post-merge. The series is **well-tested**, with 204 lines of new tests covering edge cases like self-referential upstreams and push divergence.

**Key participants**: Harald Nordgren, Junio C Hamano, Phillip Wood.
**Status**: **Ready for `next`**; expected in an upcoming "What's cooking" report.

---

### **`git cat-file --batch-command` remote-object-info graduates to `next`**
Pablo Sabater’s **21-patch series** enabling `git cat-file --batch-command remote-object-info` was **merged to `next`** after Junio’s explicit confirmation. The feature allows Git clients to query object metadata (size, type) from remote repositories without downloading full objects, using a new `object-info` protocol v2 capability. The v21 revision addressed the last cosmetic nits (documentation, macro formatting), and the series is **structurally unchanged** from v20, which resolved all technical concerns (dynamic capability negotiation, memory safety, server-side consistency). The implementation is **robust and well-tested**, with 747 lines of new tests covering all smart transports and edge cases.

**Key participants**: Pablo Sabater, Karthik Nayak, Junio C Hamano.
**Status**: **Merged to `next`**; on track for integration into `master`.

---

### **ODB alternates performance regression sparks architectural redesign**
A **12% performance regression in `git receive-pack`** (server-side pushes), introduced by Patrick Steinhardt’s ODB abstraction rework, **sparked a fundamental architectural proposal**. The regression stems from a shift in object lookup order from "pack-first, loose-second" *per backend* to "pack-loose, pack-loose" *across backends*, causing redundant loose object checks in alternates directories. **Jeff King (Peff)** reframed the issue as a **systemic alternates problem**, while Patrick proposed moving alternates handling into the "files" backend itself, treating them as an implementation detail. Peff endorsed the redesign, dismissing concerns about lost flexibility as "unlikely to be practical." The proposal would enable pack-first-loose-second lookups across all alternates while simplifying commit-graph/bitmap loading. A near-term fix (targeted lookup order adjustment) may still land before Patrick’s redesign, planned for late July.

**Key participants**: Patrick Steinhardt, Jeff King, Junio C Hamano, Wolfgang Kritzinger (Atlassian).
**Status**: **Architectural proposal under discussion**; near-term fix possible.

---

### **`diff.<driver>.process` RFC reaches technical completion**
Michael Montalbo’s **9-patch RFC** for external diff/blame tool integration **reached technical completion** after resolving its last merge conflict. The series introduces a **long-running subprocess protocol** that lets tools inject hunks into Git’s pipelines while preserving downstream features (word diff, `--stat`, `git blame`, `git log -L`). The v6 revision wired `--stat`/`--numstat` to the external process and completed the integration for `git log -L`. Junio’s final sign-off confirmed the series is unblocked; the only remaining cases using Git’s builtin diff are combined diffs (`--cc`). The protocol’s design—**forward-compatible, cache-aware, and robust to tool failures**—sets a new standard for external tool integration.

**Key participants**: Michael Montalbo, Junio C Hamano, Eric Sunshine.
**Status**: **Ready for `next`**; expected in an upcoming "What's cooking" report.

---

### **`git mv` misleading error messages fixed**
Lucas Zamboni Orioli’s **2-patch series** fixing misleading `git mv` errors (e.g., blaming the source file when the destination directory is missing) **cleared its last design hurdle**. Junio’s final review approved the use of `has_symlink_leading_path()` to reject symlinks in the destination path, aligning with `git apply` precedent. The series improves error messages to include both source and destination paths, adds an early existence check for the destination directory, and includes **83 new test lines** covering symlink edge cases. The fix is **user-facing and low-risk**, addressing a long-standing usability issue.

**Key participants**: Lucas Zamboni Orioli, Junio C Hamano, Ben Knoble.
**Status**: **Ready for `next`**.

---

### **`git rebase --update-refs` symref bugfix nears merge**
Son Luong Ngoc’s **2-patch series** fixing edge cases with symbolic references in `git rebase --update-refs` **cleared its last design hurdle**. Junio’s final review approved the use of `has_symlink_leading_path()` to reject tracked symlinks in the destination path. The series handles **non-branch symrefs** (e.g., tags) and **cross-worktree conflicts**, with **83 new test lines** covering symlinks as both final and intermediate components. The fix addresses a **real-world pain point** for branch rename migrations and multi-worktree setups.

**Key participants**: Son Luong Ngoc, Junio C Hamano, Phillip Wood.
**Status**: **Ready for `next`**.

---

### **Memory-safety bug in `git worktree add` can cause data loss**
Matthias Aßhauer posted a **critical bugfix** for `git worktree add` that prevents an out-of-bounds read leading to **silent data corruption** (recursive deletion of the working directory, including `.git`). The bug surfaces when an empty string or a string of only directory separators (e.g., `""` or `"//"`) is passed as the worktree path, likely via an unset environment variable. Junio’s review identified a **logical flaw** in the proposed fix (potential pointer underflow), and a v2 is expected soon. The series is **under active review** and high-priority.

**Key participants**: Matthias Aßhauer, Junio C Hamano.
**Status**: **Bugfix in progress**; v2 expected.

---

## In brief

**`git history squash` reaches v10, ready for integration** -- Harald Nordgren’s **`git history squash`** subcommand, which folds commit ranges into a single commit while preserving descendant history, **reached its tenth iteration** and is **functionally complete**. Junio’s "Will replace" sign-off on v7 signals intent to queue the series for the next release. The command avoids the **repeated conflict stops** of a rebase-based approach, making it ideal for **automated workflows** and **interactive debugging**.
**Status**: **Ready for `next`**.

**Packfile URI race conditions: test flakiness blocks progress** -- Ted Nyman’s v4 series fixing race conditions in Git’s HTTP transport during concurrent packfile URI and dumb HTTP downloads remains **blocked by test flakiness**. Jeff King identified a **false positive** in the `--stress` test, and the fix is to **drop the `test -s` check**. The series is otherwise **technically complete**.
**Status**: **Blocked by test flakiness**; v5 expected.

**`git send-email` filesystem path completion** -- Yury Norov’s patch to extend bash completion for `git send-email` to include filesystem paths remains **under review**, with a **usability trade-off unresolved**. SZEDER Gábor argues that offering both refs and patch files in the same completion list degrades usability, while Junio notes this would make the feature useless for his workflow.
**Status**: **Under review**; v2 expected.

**`git repo info` path keys** -- K Jayatheerth’s **7-patch series** adding path-related keys (e.g., `path.toplevel`, `path.hooks`) to `git repo info` is now **mechanically clean and ready for substantive review**. Junio’s acknowledgment signals openness to the design.
**Status**: **Ready for review**.

**`git stash rename`** -- User testimonials reinforced the justification for the feature, but **Junio’s skepticism persists** about the current stash design. No progress on implementation.
**Status**: **Stalled**.

**CI toolchain upgrades** -- Jeff King’s **2-patch series** bumping the Ubuntu image for static-analysis jobs to `ubuntu-24.04` (picking up Coccinelle 1.3.0+) received **Junio’s approval**. The series also includes a **preparatory cleanup** in `bloom.c` to silence a Coccinelle false positive.
**Status**: **Ready for integration**.

**`git config --url` gains scp-style URL support** -- Fabian Pottbäcker’s **3-patch series** adding scp-style SSH URL support (e.g., `host:path`) to `git config --url` is **uncontroversial and ready for review**.
**Status**: **Ready for review**.

**`git clone --revision` segfault fix** -- Adrian Friedli’s **one-line NULL check** to fix a segfault when using `--revision` with servers that advertise more refs than requested has been **merged to `master`**.
**Status**: **Merged**.

**UTF-8 string-width refactoring** -- Hardik Kumar’s **v2 patch** changing `utf8_strwidth()` to return `size_t` addressed reviewer concerns by updating all callers. The **overflow check removal** remains unresolved.
**Status**: **Under review**.

---

## Looking ahead

### **Topics likely to dominate next week**
- **ODB alternates redesign**: Patrick Steinhardt’s **architectural shift** (moving alternates into the "files" backend) is the most consequential in-flight work, with **Peff’s endorsement** boosting its credibility. A near-term fix (targeted lookup order adjustment) may land before the redesign.
- **`git stash` redesign**: Junio’s suggestion to **tie stashes to branches** could subsume the `rename` feature, but no concrete proposals yet.
- **Packfile URI race conditions**: Ted Nyman’s series is **blocked by test flakiness**, but the fix is straightforward (drop the `test -s` check). The core logic is sound, and the series is otherwise ready for `next`.
- **`git worktree add` memory-safety bug**: Matthias Aßhauer’s bugfix is **high-priority**, with a v2 expected soon.

### **Ongoing efforts to watch**
- **Rustification**: No updates; the effort remains **contentious** (Randall S. Becker’s NonStop port concerns vs. memory safety goals).
- **`paint_down_to_common()` optimization**: Elijah Newren’s **resumed review** of the technical document surfaced precision gaps around v1 commit-graph handling. Kristofer Karlsson proposed a `commit_graph_generation_topo_ceiling()` helper to clarify invariants.
- **`git maintenance` gc-promisor task**: The proposed `git maintenance run --task=gc-promisor` subcommand remains under design discussion, with performance testing on large promisor packs underway.