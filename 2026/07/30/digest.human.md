# The Git Project Daily Digest - July 30, 2026

## The day in brief

A busy day on the Git mailing list (73 emails across 20 threads) saw **two major series reach readiness for merging**, documentation improvements graduate to `next`, and **substantive design debates** about new features. The `git branch --delete-merged` series from Harald Nordgren reached its 24th iteration with all blocking issues resolved, while Lucas Zamboni Orioli's `git mv` symlink-handling fix received final maintainer approval. Documentation efforts continued to bear fruit, with Kristoffer Haugsbakk's `git-interpret-trailers` overhaul now cooking in `next`. Design discussions about `git add --resolved` and `git repack --drop-filtered` revealed philosophical divides about Git's interface conventions.

---

## Notable threads

### **`git branch --delete-merged` reaches final form (v24)**
**Headline:** Harald Nordgren's long-running series adding `--delete-merged` to `git branch` is now **implementation-complete and ready for merging**, with all blocking correctness issues resolved in v24.

The series introduces safe automated local branch cleanup with comprehensive filtering and safety controls. The v24 update fixes two critical issues identified by Phillip Wood: (1) **corrected push-detection logic** in `branch_pushes_to_upstream()` to accurately determine whether pushing a branch would update its upstream, and (2) **simplified stacked-branch protection** that now clears upstream configs for kept (merged) branches whose own upstream is being deleted. The implementation supports repeatable `--delete-merged` arguments (e.g., `--delete-merged origin/main --delete-merged origin/next`) and optional positional patterns (e.g., `topic-*`) to limit deletion scope.

Junio C Hamano has confirmed the series is ready for merging pending final review of the v24 fixes. The only agreed follow-up work is extending the stacked-branch protection to `git branch -d` for consistency, which will be implemented after the series lands.

**Files touched:** `builtin/branch.c`, `ref-filter.c`, `Documentation/git-branch.adoc`, `Documentation/config/branch.txt`, `t/t3200-branch.sh`

---

### **`git mv` symlink-handling fix receives final approval**
**Headline:** Lucas Zamboni Orioli's two-patch bugfix series for `git mv` has received **final maintainer approval** after addressing Junio C Hamano's requested refactoring.

The series fixes misleading error messages when the destination's parent directory doesn't exist or contains problematic components, and adds an early check to reject tracked symlinks in the destination path. The v5 update extracted shared logic into a `needs_worktree_rename()` helper function, ensuring consistency between the early check and the actual rename operation. Junio's "Looking good" signals the series is ready for `next`.

The fix addresses a long-standing usability issue where `git mv` would blame the source file for destination directory problems, and prevents index corruption when moving through symlinks. The implementation uses `has_symlink_leading_path()` (borrowed from `git apply`) to avoid following tracked symlinks, aligning with Git's policy of not traversing them.

**Files touched:** `builtin/mv.c`, `t/t7001-mv.sh`

---

### **Documentation efforts graduate to `next`**
**Headline:** Two documentation series from Kristoffer Haugsbakk have been **revived from "Will discard" status** and are now cooking in `next`.

1. **`git-interpret-trailers` overhaul (v4):** An 11-patch series that comprehensively overhauls the man page to clarify terminology, explain the strict key format (ASCII alphanumeric + hyphen only), and document how comment lines are handled. The series addresses a prior bug report about silent failures when parsing keys containing periods (e.g., "My-6.11-Version") and introduces consistent terms like "trailer block" and "metadata." Junio's "What's cooking" report now lists the topic as cooking in `next`.

2. **`git replay` config documentation (v4):** A four-patch series that synchronizes `replay.refAction` documentation between `git-config(1)` and `git-replay(1)` man pages. The series implements Git documentation conventions through conditional linking, description lists, and consistent default value placement. Junio has replaced v3 with v4 in the integration queue after addressing minor phrasing tweaks.

Both series are **documentation-only** and address real usability gaps identified by contributors. Their graduation to `next` reflects the project's ongoing investment in improving documentation clarity and consistency.

---

### **Design debate: `git add --resolved` vs `--unmerged`**
**Headline:** A **philosophical divide** has emerged in the design of `git add --resolved`, with Junio C Hamano defending the workflow-oriented `--resolved` option while Michael Montalbo proposes a more general `--unmerged` selector with policy flags.

Junio's implementation introduces `--resolved` as a monolithic operation that stages only paths whose conflict markers have been removed, arguing this aligns with Git's existing conventions (e.g., `git am --resolved`) and reflects user intent. The series consolidates duplicate conflict-marker detection logic and adds helpers for index removal with flags like `--dry-run` and `--verbose`.

Montalbo's critique centers on the option name, arguing it conflates selection (unmerged paths) with policy (conflict-marker check). He proposes an alternative design: a general `--unmerged` selector with explicit policy flags like `--allow-conflict-markers` or `--skip-conflict-markers`. This approach would decouple the selection of unmerged paths from the policy about whether to stage them, offering more flexibility.

The debate highlights a tension between Git's traditional workflow-oriented commands and a more modular approach that exposes underlying implementation details. No consensus has been reached, and the discussion may continue with further arguments or counter-proposals.

**Files touched (proposed):** `merge-ll.c`, `merge-ll.h`, `read-cache.c`, `read-cache-ll.h`, `builtin/add.c`, `diff.c`, `rerere.c`, `Documentation/git-add.txt`, `t/t2207-add-resolved.sh`

---

### **`git repack --drop-filtered` for partial clones (RFC v2)**
**Headline:** Siddharth Shrimali's RFC v2 series introducing `--drop-filtered` to `git repack` for partial clones is now **ready for substantive review** after addressing all blocking issues from v1.

The feature allows users to safely reclaim disk space in partial clones by removing locally cached promisor blobs exceeding a user-specified size threshold (e.g., `--filter=blob:limit=10M`). The v2 update fixes the `--write-bitmap-index` validation flaw, implements critical safety guards (merge/rebase/cherry-pick checks and index validation), and adds comprehensive documentation.

The implementation adds two new options: `--drop-filtered` (real runs) and `--dry-run` (reporting only). It introduces `repack-filtered.c` for the core logic, extends `list-objects-filter` with `list_objects_filter__filter_oidset()`, and modifies `repack-promisor.c` to support excluding objects during promisor pack rebuilding. Safety is guaranteed by construction: only objects marked as promisor (recoverable from the remote) are targeted, and locally created objects are untouched.

The series is well-structured and addresses a real pain point for partial clone users. Reviewers with expertise in the repack machinery (e.g., Patrick Steinhardt, Christian Couder, or Taylor Blau) are likely to weigh in. The deferred work (drop log, remote verification) is explicitly scoped and does not block the core functionality.

**Files touched:** `builtin/repack.c`, `repack-filtered.c`, `repack-promisor.c`, `list-objects-filter.{c,h}`, `Documentation/git-repack.adoc`, `t/t7706-repack-drop-filtered.sh`

---

## In brief

**`git rebase --update-refs` symref bugfix:** Son Luong Ngoc's series addressing edge cases with branch symrefs in `git rebase --update-refs` continues to progress, with Phillip Wood clarifying a detail about TopGit's ref naming conventions. The v4 iteration is expected to address the remaining correctness issues.

**`git history squash` design discussion:** Harald Nordgren proposed removing the `--reedit-message` option from `git history squash` and making re-editing the default behavior. The proposal does not address Phillip Wood's broader correctness concerns about `UNINTERESTING` commits, root commits, or multi-tip histories.

**ODB abstraction series:** Toon Claes provided substantive review of Patrick Steinhardt's ODB abstraction series, elevating the first patch's importance from a mechanical cleanup to a **latent correctness bug** affecting repositories with alternates and compatibility hash algorithms. The review includes a concrete test case to verify the fix.

**`git add --resolved` test helper:** Shlok Kulshreshtha's two-patch series introducing the `commit_body()` test helper has been marked for `next`. The helper replaces 61 exit-code-masking instances of `git cat-file commit <rev> | sed -e '1,/^$/d'` with a safer, reusable shell function.

**`git merge-base --is-ancestor` tests:** Nikolaus Schuetz's test coverage for `git merge-base --is-ancestor` has been queued for integration. The v2 revision reuses the existing test repository history and adds a test case for rejecting excess arguments.

**`git maintenance` regression fix:** David Lin's patch fixing a regression in `git maintenance`'s "none" strategy handling received surface-level review from Junio, who requested an in-body `From:` line to align author and sign-off addresses.

**Hex object ID case sensitivity (RFC):** brian m. carlson's RFC series proposing to restrict hex object IDs to lowercase only in Git 3.0 continues to face pushback from Junio. brian provided concrete evidence of real-world security vulnerabilities caused by Git's current case-insensitive parsing, but Junio remains unconvinced that the benefit outweighs the disruption.

**`git stash` time-based selector retraction:** Junio C Hamano retracted his two-patch series that proposed rejecting time-based stash selectors in `git stash drop` and `git stash pop` after discovering the reflog expiration machinery already limits removal to a single entry.

**`--date=elapsed` for Git:** Israel Roldan proposed a new `--date=elapsed` (and `--date=elapsed-pad`) date format for Git, along with corresponding `%aX` and `%cX` pretty-format placeholders. The feature provides a precise, script-friendly elapsed-time format (e.g., "1y 11M 5d 13h 5m 13s") modeled after `uptime(1)`.

**`git config --global` documentation discrepancy:** Nils Fahldieck reported a bug where `git config --global` reads only one file despite the documentation claiming it reads both `~/.gitconfig` and the XDG config file. Junio agreed the documentation is incorrect but has not endorsed a specific fix.

---

## On the radar

**`git history squash`:** Phillip Wood is actively developing fixups to address the reachability logic flaws in `hn/history-squash`. The topic remains in `seen` pending these updates.

**`git repack --drop-filtered`:** Siddharth Shrimali's RFC v2 series is ready for substantive review. The core functionality is complete, and the deferred work (drop log, remote verification) is explicitly scoped.

**ODB abstraction:** Patrick Steinhardt's series making on-disk ODB structures pluggable continues to receive substantive review. The first patch is now recognized as fixing a latent correctness bug affecting repositories with alternates and compatibility hash algorithms.

**Hex object ID case sensitivity:** The RFC series proposing to restrict hex object IDs to lowercase only in Git 3.0 remains under discussion. brian m. carlson has provided concrete evidence of real-world security vulnerabilities, but Junio remains unconvinced that the benefit justifies the disruption.