# Git Mailing List Digest: 2026/07/20 -- 2026/07/26

## The period in brief
This was a **highly productive week** on the Git mailing list, with **353 emails across 109 threads** covering seven days. Traffic was **heavy and technically dense**, dominated by **long-running refactoring efforts**, **performance regressions**, and **feature finalizations**. The most consequential developments: **`git branch --delete-merged` and `git cat-file --batch-command remote-object-info` reached technical completion**, while **ODB alternates handling and `diff.<driver>.process` integration** sparked architectural debates. Smaller but impactful fixes (e.g., `git mv` error messages, `git worktree add` memory safety) also landed. The tone was **collaborative but occasionally contentious**, particularly around the `lib/` directory reorganization and ODB performance trade-offs.

---

## Key developments

### **`git branch --delete-merged` lands after 23 iterations**
Harald Nordgren’s **`git branch --delete-merged`** series, now in its 23rd iteration, **reached technical completion and is ready for merging**. The command provides **safe, automated cleanup of local branches** that have been merged into a specified upstream, with **comprehensive safety controls**:
- **Order-independent stacked-branch protection**: Fails to delete a branch if it is used as an upstream for an unmerged branch, but clears the upstream config for kept (merged) branches whose own upstream is being deleted.
- **Per-branch opt-out**: The `branch.<name>.deleteMerged=false` config key exempts specific branches from automated cleanup.
- **`--dry-run` preview**: Prints "Would delete" or "Would skip" messages without modifying refs or config.
- **Repeatable upstream selectors**: Supports multiple `--delete-merged` arguments (e.g., `--delete-merged origin/main --delete-merged origin/next`) and optional positional patterns (e.g., `topic-*`).

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), Phillip Wood (reviewer).
**Status**: **Ready for `next`**; the only agreed follow-up is extending stacked-branch protection to `git branch -d` for consistency.

---

### **`git cat-file --batch-command remote-object-info` graduates to `next`**
Pablo Sabater’s **21st iteration** of the `git cat-file --batch-command remote-object-info` series was **merged to `next`**, enabling **efficient remote object metadata queries** (e.g., size, type) without downloading objects. The series introduces a new `object-info` protocol v2 capability with **dynamic capability negotiation**, **memory safety**, and **robust edge-case handling** (10,000-object batch limit, 8KB URL length cap). The implementation is **security-hardened**, with 747 lines of new tests covering all smart transports (`git://`, `file://`, `http://`).

**Key participants**: Pablo Sabater (author), Karthik Nayak (reviewer), Junio C Hamano (maintainer).
**Status**: **Cooking in `next`**; on track for integration into `master` in a future release.

---

### **ODB alternates performance regression sparks architectural debate**
A **12% performance regression in `git receive-pack`** (server-side pushes) introduced by the ODB abstraction rework sparked a **substantive architectural discussion**. The regression, reported by Wolfgang Kritzinger (Atlassian), stems from a shift in object lookup order from **"pack-first, loose-second" *per backend*** to **"pack-loose, pack-loose" *across backends***, causing redundant loose object checks in earlier backends (e.g., quarantine directories).

**Proposed solutions**:
1. **Near-term fix**: Restore the "pack-first, loose-second" *across backends* strategy to eliminate redundant loose object checks (Junio C Hamano).
2. **Long-term redesign**: Move alternates handling into the "files" backend itself, treating alternates as an implementation detail (Patrick Steinhardt). This would enable optimizations like pack-first-loose-second lookups across all alternates while simplifying other subsystems (commit-graph, bitmap, maintenance).

**Key participants**: Patrick Steinhardt (author), Jeff King (Peff), Junio C Hamano (maintainer), Wolfgang Kritzinger (reporter).
**Status**: **No resolution yet**; Patrick’s redesign is planned for late July, but a targeted fix may land sooner.

---

### **`diff.<driver>.process` RFC reaches technical completion**
Michael Montalbo’s **9-patch RFC** for external diff/blame tool integration reached **"technically complete and ready for merging"** status. The series introduces a **long-running subprocess protocol** that lets tools inject hunks into Git’s pipelines while preserving downstream features (word diff, `--stat`, `git blame`, `git log -L`). The protocol is **forward-compatible, cache-aware, and robust to tool failures**, setting a new standard for external tool integration.

**Key features**:
- **End-to-end integration**: Tools can now participate in `--stat`, `--numstat`, and `git log -L` operations.
- **Consistency**: Ensures diff output aligns with the tool’s hunks (not Git’s builtin diff).
- **Safety**: Handles tool failures gracefully, falling back to Git’s builtin diff.

**Key participants**: Michael Montalbo (author), Junio C Hamano (maintainer).
**Status**: **Ready for `next`**; the only remaining cases using Git’s builtin diff are combined diffs (`--cc`).

---

### **`git mv` misleading error messages fixed**
Lucas Zamboni Orioli’s **2-patch series** fixing misleading `git mv` errors (e.g., blaming the source file when the destination directory is missing) is now **ready for integration**. The series improves error messages to include both source and destination paths and adds an early existence check for the destination directory, rejecting **missing directories, non-directory components, and tracked symlinks**.

**Key participants**: Lucas Zamboni Orioli (author), Junio C Hamano (maintainer), Ben Knoble (reviewer).
**Status**: **Ready for `next`**; includes 83 new lines of test coverage in `t/t7001-mv.sh`.

---

### **Memory-safety bug in `git worktree add` can cause data loss**
Matthias Aßhauer posted a **critical bugfix** for `git worktree add` that prevents an out-of-bounds read leading to **silent data corruption** (recursive deletion of the working directory, including `.git`). The bug surfaces when an empty string or a string of only directory separators (e.g., `""` or `"//"`) is passed as the worktree path, likely via an unset environment variable.

**Key participants**: Matthias Aßhauer (author), Junio C Hamano (maintainer).
**Status**: **Under active review**; a v2 is expected to address a logical flaw in the proposed fix.

---

## In brief

**`git rebase --update-refs` symref bugfix** -- Son Luong Ngoc’s **2-patch series** fixing edge cases with symbolic references in `git rebase --update-refs` is **functionally complete**, with only minor test organization tweaks requested. The series addresses a latent bug where symrefs (e.g., `main` pointing to `master`) cause rebase failures during ref updates.

**Packfile URI race conditions** -- Ted Nyman’s **v4 series** fixing race conditions in Git’s HTTP transport during concurrent packfile URI and dumb HTTP downloads is **blocked by test flakiness**, despite the core logic being sound. The fix preserves resumable downloads by using deterministic staging paths.

**`git repo info` path keys** -- K Jayatheerth’s **7-patch series** adding path-related keys (e.g., `path.toplevel`, `path.hooks`) to `git repo info` is **mechanically clean and ready for substantive review**. The design aligns with `git rev-parse --git-path` precedent.

**CI toolchain upgrades** -- Jeff King’s **2-patch series** bumping the Ubuntu image for static-analysis jobs to `ubuntu-24.04` (picking up Coccinelle 1.3.0+) received **Junio’s approval**. The series also includes a preparatory cleanup in `bloom.c` to silence a Coccinelle false positive.

**`git stash rename`** -- User testimonials reinforced the justification for the feature, but **Junio’s skepticism persists** about the current stash design. No progress on implementation.

**`paint_down_to_common()` optimization** -- Elijah Newren’s **resumed review** surfaced precision gaps around v1 commit-graph handling. Kristofer Karlsson proposed a `commit_graph_generation_topo_ceiling()` helper to clarify invariants.

**`git send-email` filesystem path completion** -- Yury Norov’s patch to extend bash completion for `git send-email` to include filesystem paths remains **under review**, with a **usability trade-off unresolved** (offering both refs and patch files in the same completion list).

**`git repack --drop-filtered`** -- Siddharth Shrimali and Siddharth Asthana agreed that **safety guards** (preventing drops during merges/rebases/cherry-picks and refusing to drop blobs referenced by the current index) are **non-negotiable for v2**.

**UTF-8 string-width refactoring** -- Hardik Kumar’s **v2 patch** changing `utf8_strwidth()` to return `size_t` addressed reviewer concerns by updating all callers. The **overflow check removal** remains unresolved.

---

## Looking ahead
- **ODB alternates redesign**: Patrick Steinhardt’s **architectural shift** (moving alternates into the "files" backend) is the most consequential in-flight work, with **Peff’s endorsement** boosting its credibility. A near-term fix may still land before the redesign.
- **`git stash` redesign**: Junio’s suggestion to **tie stashes to branches** could subsume the `rename` feature, but no concrete proposals yet.
- **Rustification**: No updates; the effort remains **contentious** (Randall S. Becker’s NonStop port concerns vs. memory safety goals).
- **`git maintenance` gc-promisor task**: The proposed `git maintenance run --task=gc-promisor` subcommand remains under design discussion, with performance testing on large promisor packs underway.