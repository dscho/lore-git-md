## The day in brief

**2026-07-22** was a busy day on the Git mailing list, with **46 emails across 18 threads**. The traffic was a mix of **final polish on nearly-ready series**, **substantive reviews of long-running efforts**, and **a handful of new bug reports and user requests**. Two major series—**`git cat-file --batch-command` remote-object-info** and **`git branch --delete-merged`**—reached the finish line, while a **performance regression in `git receive-pack`** sparked architectural discussion. A **macOS-specific memory leak** and a **submodule symlink opt-out request** also surfaced.

---

## Notable threads

### **`git cat-file --batch-command` remote-object-info (v20) – ready for `next`**
**Headline**: The 20th iteration of Pablo Sabater’s security-hardened `git cat-file --batch-command remote-object-info` series is now **technically complete and cleared for `next`**, with only cosmetic nits remaining.

The series enables Git clients to query object metadata (size, with type support designed for future expansion) from remote repositories without downloading the objects themselves. The v20 changes focus on **dynamic capability negotiation** (replacing a hardcoded allow-list with runtime-negotiated metadata fields) and **build fixes** (resolving signed/unsigned comparison warnings). The latest exchange between Pablo and Karthik Nayak confirms that two minor cosmetic issues—**documentation `CAVEATS` references** and **`EXPAND_DATA_INIT` macro formatting**—will **not block graduation**. Junio’s prior confirmation that no unresolved design or implementation concerns remain now stands fully validated.

The series is **self-contained, well-tested (747 new lines in `t/t1017-cat-file-remote-object-info.sh`), and security-conscious**, with strict input validation, memory safety, and protocol hardening. It is now **queued for `next`**, with the unresolved cosmetic nits to be addressed in a future cleanup if not resolved before integration.

---

### **`git branch --delete-merged` (v20) – unblocked and ready for `next`**
**Headline**: Harald Nordgren’s **`git branch --delete-merged`** series, now in its 20th iteration, has resolved its last correctness issue—**order dependency in stacked-branch protection**—and is **ready for merging**.

The series introduces a safe, automated local-branch cleanup command with **comprehensive filtering and safety controls**, including **per-branch opt-out**, **`--dry-run` preview**, and **order-independent stacked-branch protection**. The v20 refactor replaces the previous two-phase approach (marking spared branches during iteration, then clearing their upstreams afterward) with a **single-pass graph traversal** that collects all protected branches before mutating the deletion set. This addresses Junio’s blocking correctness concern and makes the behavior deterministic.

The series is **well-motivated** (addressing a real workflow pain point), **thoroughly tested** (comprehensive integration tests in `t/t3200-branch.sh`), and **designed for future extensibility** (flags-based API, composable filters). Phillip Wood’s **substantive review** of the refactor confirms the logic is sound, and the only agreed follow-up work—**extending stacked-branch protection to `git branch -d` for consistency**—will be implemented after the series lands.

The series is now **unblocked and ready for `next`**, with no known issues.

---

### **Performance regression in `git receive-pack` after ODB rework**
**Headline**: A **12% performance regression in `git receive-pack`** (server-side pushes) has been traced to the ODB abstraction rework, sparking a discussion about **short-term fixes vs. long-term architectural redesign**.

The regression was introduced by commit `a593373b09`, which shifted the ODB lookup strategy from **"pack-first, loose-second" *per backend*** to **"pack-loose, pack-loose" *across backends***. This causes redundant loose object checks in earlier backends (e.g., quarantine directories) before finding objects in packs in later backends, particularly costly on NFS-backed deployments. Peff’s reproduction case (using `GIT_ALTERNATE_OBJECT_DIRECTORIES`) demonstrates the issue without involving transactions or temporary directories.

**Short-term fix**: Junio proposes restoring the **"pack-first, loose-second" *across backends*** strategy to eliminate redundant loose object checks. This is separable from Patrick Steinhardt’s **long-term architectural redesign** (transaction-aware ODB writes), which would bypass the ODB source list for writes during active transactions. Patrick’s proposal ties the regression fix to a larger refactoring effort, delaying implementation until late July or August.

The thread highlights a tension between **immediate performance needs** and **architectural purity**, with no clear resolution yet. The regression may persist until Patrick’s redesign is ready, unless someone volunteers to implement Junio’s targeted fix in the interim.

---

### **`git rebase --update-refs` symref handling (v3) – ready for review**
**Headline**: Son Luong Ngoc’s **two-patch bugfix series** for `git rebase --update-refs` handling of symbolic references (symrefs) is now **technically complete and ready for maintainer consideration**.

The series addresses edge cases where symrefs (e.g., `main` pointing to `master` during a branch rename migration) cause rebase failures after the primary branch has already been successfully rewritten. The v3 revision splits the fix into two focused patches:
1. **Skipping branch-to-branch symrefs** to prevent duplicate updates.
2. **Safeguarding non-branch symref targets** (tags, notes) and preventing duplicate updates when multiple aliases point to the same ref.

The implementation is **backend-agnostic**, passing tests with both files and reftable backends, and includes **thorough test coverage** in `t3404-rebase-interactive.sh`. Phillip Wood’s feedback from v2 has been fully addressed, and the series appears ready for integration.

---

### **`git mv` misleading error message (v2 planned)**
**Headline**: Lucas Zamboni Orioli’s bugfix for `git mv`’s misleading error message is **planned for a v2 split**, addressing Ben Knoble’s review feedback.

The patch will be split into two commits:
1. **Error message clarity**: Make the error unambiguous by naming both source and destination paths (e.g., `renaming 'source/file' to 'destination/file' failed: No such file or directory`).
2. **Early existence check for destination directory**: Enable `--dry-run` detection, though the **TOCTOU race** remains unresolved.

The first commit is now **race-free and stands alone**, while the second remains subject to the TOCTOU concern. The l10n team is flagged for the new translatable string.

---

## In brief

**`git send-email` filesystem path completion** -- Yury Norov’s patch to extend bash completion for `git send-email` to include filesystem paths remains **under review**, with a **usability trade-off unresolved**. SZEDER Gábor argues that offering both refs and patch files in the same completion list degrades usability, while Junio notes this would make the feature useless for his workflow. A v2 is expected to reconcile these concerns.

**ODB optimizations pluggable (v2)** -- Patrick Steinhardt’s 12-patch series to make ODB optimizations pluggable is **under review**, with Junio’s feedback on v1 fully addressed. The series prepares the codebase for backend-specific housekeeping tasks (repacking, geometric repacking, auto-gc) and is **ready for broader review**.

**Loose object backend refactoring (v1)** -- Patrick Steinhardt’s nine-patch series to move loose object writing logic into `odb/source-loose.c` is **under review**, with **Toon Claes providing substantive reviews** of patches 4/9 and 9/9. The series is part of the ongoing ODB abstraction effort and is **technically sound**, though reviewers may surface questions about transaction interfaces or performance.

**`git rebase --continue --[no-]edit`** -- Hugo Sales’ patch to add `--[no-]edit` to `git rebase --continue` remains **stalled on design concerns**. Junio and Phillip Wood question the feature’s necessity and the scope of the `rebase.noEdit` config option, particularly its interaction with interactive rebase steps (`edit`, `reword`). A v2 is needed to address these edge cases.

**`git submodule` remote matching with `insteadOf`** -- Éric NICOLAS’s bugfix for `git submodule update --remote` failing to match remotes when URL aliases are used is **ready for v2**. Junio’s review requested minor clarifications to the commit message and a small code simplification.

**macOS TRE regexec() memory leak** -- Chungmin Lee’s **platform-specific bugfix** for a memory leak in Git’s macOS builds when the system regex engine processes invalid UTF-8 sequences is **under review**. The patch adds a Darwin-specific workaround that splits input buffers at invalid multibyte boundaries, avoiding the leaking code path. **Worth attention from macOS users and regex subsystem experts.**

**`git-p4` commit metadata handling** -- Hardik Kumar’s patch to clarify a 2007 `# fixme` in `git-p4.py` remains **stalled on historical ambiguity**. The thread has shifted to determining the original intent of the `# fixme`, with no progress toward resolving the core question.

**Revision walker `--exclude-first-parent-only` fix** -- Junio’s **standalone bugfix** for the revision walker’s `--exclude-first-parent-only` option is **under review**, with one `Reviewed-by` (Jerry Zhang) recorded. The patch ensures the option is honored even when the first parent has already been marked `SEEN`.

**Submodule symlinks opt-out request** -- Kyle Marek’s **user request** for a configuration knob (`safe.submodule.path`) to opt out of the security-motivated restriction on symlinks in submodule paths remains **unaddressed**. The request highlights a tension between **security and workflow flexibility**, with no patch yet proposed.

---

## On the radar

- **`cc/fast-import-usage` (stalled)**: Christian Couder is on vacation, and Junio has opted to retain the topic in his integration tree for now, absent conflicts. The series may be dropped if it blocks other work or if Christian remains unavailable.
- **`tc/replay-linearize` (on hold)**: Waiting for author response.
- **`rebase --continue --[no-]edit`**: Needs a v2 to address design concerns about config scope and edge cases.
- **ODB performance regression**: Awaiting a targeted fix or Patrick Steinhardt’s transaction-aware ODB redesign.