Here is the digest for August 4, 2026:

---

## The day in brief

August 4 was a **heavy-traffic day** (113 emails, 27 threads) with **two major milestones**: the `git branch --delete-merged` series reached **implementation-complete** status and was **merged to `next`**, while the `git history squash` series **resolved its last technical blockers** and is now **ready for v13**. A **design debate** over `post-worktree-*` hooks dominated discussion, with Junio challenging the necessity of the feature. **Performance regressions** (ODB alternates, HTTP daemon races) and **breaking changes** (hex object IDs, textconv on NonStop) also surfaced, alongside **routine refactoring** (fast-import, ODB streaming API) and **documentation cleanups**.

---

## Notable threads

### **`git branch --delete-merged` lands in `next`**
**Headline**: Harald Nordgren’s 24-iteration series implementing `git branch --delete-merged` is now **implementation-complete** and **merged to `next`**, offering safe, automated local branch cleanup with **order-independent stacked-branch protection**, **per-branch opt-out**, and **`--dry-run` preview**.

**What’s new**:
- **v24 fixes** resolved the last two blocking issues:
  1. **Push-detection logic**: `branch_pushes_to_upstream()` now correctly applies push/fetch refspecs, preserving branches like `main` tracking `origin/main` even when `remote.pushDefault` names a fork.
  2. **Stacked-branch protection**: Simplified to protect only the *direct* upstream of a surviving branch, clearing stale upstream configs for kept (merged) branches.
- **Safety mechanisms**: Checked-out branches, missing upstreams, push divergence, and `branch.<name>.deleteMerged=false` opt-out are all enforced.
- **API refactoring**: Patches 2–4/7 introduced a flags-based API (`DELETE_BRANCH_FORCE`, `DELETE_BRANCH_DRY_RUN`, etc.), eliminating local boolean caching to prevent stale values.
- **Test coverage**: Expanded to verify the push-detection fix, stacked-branch protection, and `--dry-run` behavior (which now avoids modifying config).

**Status**: **Merged to `next`**. Junio and Phillip Wood confirmed the series is ready, with only a **surface-level nit** (unwrapping long test command lines) remaining. The agreed follow-up—extending stacked-branch protection to `git branch -d`—will be implemented post-merge.

**Key files**: `builtin/branch.c`, `ref-filter.c`, `t/t3200-branch.sh`.

---

### **`git history squash` resolves design debate, ready for v13**
**Headline**: Harald Nordgren’s `git history squash` series (v12) **resolved its last technical blockers** and is now **ready for v13**, with Phillip Wood’s fixups addressing **reachability logic flaws** and a **consensus on editor behavior**.

**What’s new**:
- **v12 fixes**:
  - **Single-pass range validation**: Rejects root commits, multiple tips, and ranges with external parents using object flags (`SQUASH_SEEN`, `SQUASH_TIP`).
  - **Merge-parent preservation**: The squashed commit retains the full parent list of the original merge commit.
  - **Autosquash message handling**: Stricter rules for `squash!`/`amend!` commits (rejects unless target is the first commit in the range or `--edit` is used).
- **Editor behavior consensus**: After debate, the series will **open an editor by default** during squash operations, with `--no-edit` as the opt-out. This resolves Junio’s concerns about dynamic defaults (e.g., skipping the editor for `fixup!` commits) being too complex.
- **Phillip Wood’s fixups**: Address `UNINTERESTING` commits, root commits, and multi-tip histories. The incremental review roadmap (split the final patch into 5 logical pieces) remains valid but contingent on Harald’s engagement.

**Status**: **Ready for v13**. Junio’s procedural feedback—**slow down rerolls to avoid invalidating reviews**—remains a hurdle, but the technical path forward is clear.

**Key files**: `builtin/history.c`, `sequencer.c`, `t/t3455-history-squash.sh`.

---

### **`post-worktree-*` hooks: Junio challenges necessity**
**Headline**: Domen Kožar’s v2 series introducing `post-worktree-add`, `post-worktree-remove`, and `post-worktree-move` hooks faced **fundamental pushback** from Junio, who questioned whether the feature is necessary at all.

**What’s new**:
- **Junio’s critique**: Most use cases (e.g., running custom logic after `git worktree add`) can be achieved with **user-written shell wrappers** (e.g., `git-wt`). He argues that none of Git’s five valid hook use cases (countermands, post-data generation, remote access, mutual exclusion, outcome-dependent execution) apply here, as worktree lifecycle events are locally observable.
- **Domen’s rebuttal**: The hooks enable **reliable automation** for high-velocity tooling (e.g., AI-driven development environments) that create/destroy worktrees frequently. The v2 series simplifies hook signatures (e.g., `post-worktree-add` runs in the new worktree with no arguments) and adds `post-worktree-move`.
- **Design**: Hooks run **after** the event, cannot affect the outcome, and are **opt-in per command**. The `post-worktree-remove` hook runs during `git worktree prune`, ensuring coverage for manual deletions.

**Status**: **Stalled on design**. Junio’s objection shifts the burden of proof to Domen to justify the feature’s necessity. The thread may pivot to a **meta-discussion** about when hooks are appropriate in Git.

**Key files**: `builtin/worktree.c`, `t/t2403-worktree-move.sh`.

---

### **ODB alternates regression: architectural redesign proposed**
**Headline**: A **performance regression** in `git receive-pack` (12% slowdown on fast filesystems, worse on NFS) introduced by the ODB abstraction rework (`a593373b09`) sparked a **proposal to move alternates handling into the "files" backend**.

**What’s new**:
- **Root cause**: The ODB rework changed the lookup order from **"pack-first, loose-second per backend"** to **"pack-loose, pack-loose across backends"**, causing redundant loose object checks in quarantine/alternate directories before finding objects in packs.
- **Patrick Steinhardt’s proposal**: Move alternates handling into the "files" backend, treating them as an **implementation detail** rather than a cross-backend concern. This would enable **pack-first-loose-second lookups across all alternates** while simplifying commit-graph/bitmap loading.
- **Endorsements**: Peff and Justin Tobler agree this is a **reasonable course correction** for the ODB abstraction, dismissing concerns about lost flexibility (e.g., mixing alternates with non-files backends) as unlikely to be practical.
- **Justin’s parallel plan**: Decouple transaction handling from the ODB source list, eliminating the need to reorder sources during transactions.

**Status**: **Design phase**. Patrick plans to implement the redesign in late July, but no near-term fix is expected. The regression affects all alternates-based operations (`clone -s`, `--reference`, hosting deployments).

**Key files**: `odb.c`, `setup.c`.

---

### **Diff provider interface RFC: design feedback requested**
**Headline**: Michael Montalbo’s RFC for a **diff provider interface** (`diff-provider.h`) seeks feedback on a framework for consulting alternate diff sources (e.g., caches, external processes) before falling back to Git’s builtin xdiff.

**What’s new**:
- **Motivation**: The RFC aims to **unify diff/blame integration** under a single interface while enabling future extensions (e.g., language-aware diffing, AI-assisted tools). Current scope focuses on hunk headers (coordinates), with content-carrying requests deferred.
- **Providers implemented**:
  - **In-process cache** (`diff-hunks` store): Delivers 1.26–1.9× speedups for warmed caches (e.g., `git blame` on `git.git`).
  - **External process driver** (`diff.<driver>.process`): Enables authoritative answers (e.g., from language servers) via a pkt-line protocol.
- **Phillip Wood’s feedback**:
  - **Clarify strategic value**: What new capabilities does this unlock (e.g., `git blame` with syntax-aware diffing)?
  - **Terminology**: Define "authoritative," "coordinates," and "trim-stable pairs."
  - **Cache keying**: Storing `xdl_opts` directly in the cache key could break if Git’s internal representation changes.
- **Status**: **RFC stage**. The series is **technically complete** but needs sharper framing to justify its complexity. If accepted, it will return as separate series for the interface, cache, and external process provider.

**Key files**: `diff-provider.c`, `diff-hunks.c`, `t/t4220-diff-hunks.sh`.

---

## In brief

- **`git repack --drop-filtered` (RFC v2)**: Siddharth Shrimali’s series to reclaim disk space in partial clones by dropping promisor blobs exceeding a size threshold **gained reviewer endorsement** (Siddharth Asthana). The series is **feature-complete**, with safety guards (merge/rebase checks, index validation) and documentation. Junio’s proposed **time-based heuristic** (avoid culling recently fetched objects) remains under discussion.
  **Files**: `builtin/repack.c`, `t/t7706-repack-drop-filtered.sh`.

- **`git fast-import` libification**: Christian Couder’s 12-patch series to **standardize usage strings** and **reduce global variables** is **ready for review**. The final patch (11/12) removes support for undocumented options (`--alias`, `--get-mark`) on the command line.
  **Files**: `builtin/fast-import.c`, `Documentation/git-fast-import.adoc`.

- **`git last-modified` Bloom filter optimization**: Toon Claes’s series to reuse Bloom filters for `git last-modified` **resolved its last open question** (Junio’s maintainability concern about `bloom_filter_settings`). The series is **ready for reroll**.
  **Files**: `revision.c`, `t/t1503-rev-parse-verify.sh`.

- **ODB streaming API unification**: Patrick Steinhardt’s 7-patch series to merge `odb_write_stream` and `odb_read_stream` into a single `odb_stream` **hit a critical bug** in the in-memory backend (Junio). The fix will require propagating the `stream->type` parameter into `hash_object_file()`.
  **Files**: `odb/streaming.c`, `t/unit-tests/u-odb-inmemory.c`.

- **`git maintenance` "none" strategy fix**: David Lin’s patch to restore the documented `"none"` strategy handling **received substantive review** (Patrick Steinhardt). The fix is **ready to merge** pending a trivial author identity alignment.
  **Files**: `builtin/gc.c`, `t/t7900-maintenance.sh`.

- **Hex object IDs in Git 3.0**: brian m. carlson’s RFC to enforce lowercase-only hex object IDs **gained tentative acceptance** from Junio ("Will queue"), but the maintainer expressed discomfort with the philosophical implications. The thread remains in **RFC limbo**.
  **Files**: `hex.c`, `Documentation/BreakingChanges.adoc`.

- **`git-refs(1)` documentation cleanup**: Kristoffer Haugsbakk’s two-patch series to reorganize a ref migration warning into an admonition block **was approved and queued**.
  **Files**: `Documentation/git-refs.txt`.

- **`git interactive` command**: shrimech’s v3 patch proposing a new built-in `git interactive` command **faced skepticism** about its necessity (Michael Montalbo). The series remains **under review**.
  **Files**: `builtin/interactive.c`, `t/t0013-interactive.sh`.

- **Bash completion for `git diff`**: Junio’s patch to fix path completion with tracked files in subdirectories **was queued in `seen`**.
  **Files**: `contrib/completion/git-completion.bash`, `t/t9902-completion.sh`.

- **`gitk` custom commands**: Tim Wiederhake’s patch adds **user-configurable custom commands** to `gitk`’s context menus, supporting placeholder substitution (commit ID, author, etc.).
  **Files**: `gitk`.

---

## On the radar

- **`git replay` and `git history` silently drop notes**: D. Ben Knoble reported that `git replay --onto=...` and `git history fixup` **lose notes** during history rewriting. No maintainer response yet.
- **Textconv regression on NonStop**: Randall S. Becker reported that **textconv filters receive empty temp files** for binary files on NonStop since Git 2.50. The issue appears **platform-specific** (D. Ben Knoble sees populated temp files on Git 2.53.0).
- **`git worktree add` regression**: Yoichi NAKAYAMA’s patch to fix a regression where `-b`/`--B` options were ignored during DWIM **needs refinement** to avoid suppressing `HEAD`-related warnings (Junio).
- **`git cat-file --batch-command` type support**: Pablo Sabater’s GSoC series to add `%(objecttype)` support to remote-object-info queries is **feature-complete** and **merge-ready**.
  **Files**: `protocol-caps.c`, `t/t1017-cat-file.sh`.