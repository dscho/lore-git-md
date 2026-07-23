## The day in brief

**2026-07-22** was a **heavy but focused** day on the Git mailing list, with **46 emails across 18 threads** touching on **regression fixes, ODB abstraction, submodule handling, and usability debates**. The standout developments:

- **`git branch --delete-merged` (v20)** finally unblocked after resolving its last correctness issue (order-independent stacked-branch protection) and is **ready for `next`**.
- **`git cat-file --batch-command` remote-object-info (v20)** cleared its last cosmetic hurdles and is **queued for `next`**, with the maintainer’s blessing.
- **`git rebase --update-refs` symref handling (v3)** landed its final fixes for edge cases and is **technically complete**.
- **ODB abstraction** saw **substantive reviews** on loose-object backend refactoring, with Toon Claes validating the self-containment of the loose backend.
- **`git send-email` completion** remains in a **usability stalemate** between shorter completion lists (SZEDER Gábor) and workflow needs (Junio).
- **`git mv`** and **`git rebase --continue`** both saw **design clarifications** that will shape their next iterations.
- **A macOS regex memory leak** was fixed with a **platform-specific workaround**, and **push-tracking after remote rename (v4)** received **maintainer approval**.
- **`git-p4`** and **`git submodule`** threads revealed **historical ambiguities** that now block progress.

The day’s texture was **technical depth over volume**, with several threads reaching **resolution** while others stalled on **design philosophy** or **historical context**. The ODB abstraction effort continues to dominate the refactoring landscape, while usability debates (e.g., `git send-email` completion) highlight the tension between **principle and pragmatism**.

---

## Notable threads

### `git branch --delete-merged` (v20) unblocked and ready for `next`
**Headline**: Harald Nordgren’s **20-iteration series** finally resolves its last correctness issue (order-independent stacked-branch protection) and is **cleared for integration**.

The series introduces a **safe, automated local-branch cleanup** command with **per-branch opt-out**, `--dry-run` preview, and **repeatable upstream selectors**. The v20 refactor replaces the previous two-phase approach with a **single-pass graph traversal** that collects all protected branches before mutating the deletion set, ensuring consistent results regardless of ref iteration order. This addresses Junio C Hamano’s blocking concern and makes the behavior deterministic.

**Key details**:
- **Stacked-branch protection**: Fails to delete a branch if it is used as an upstream for an unmerged branch, but clears the upstream config for kept (merged) branches whose own upstream is being deleted.
- **Per-branch opt-out**: `branch.<name>.deleteMerged=false` exempts specific branches from automated cleanup.
- **`--dry-run`**: Prints "Would delete branch <name>" without modifying refs or config.
- **Test coverage**: Comprehensive integration tests verify all safety checks, edge cases (e.g., self-referential upstreams), and the new order-independent behavior.

**Status**: **Ready for `next`**. The only follow-up work is extending stacked-branch protection to `git branch -d` for consistency, which will be implemented after the series lands. Phillip Wood’s **substantive review** confirmed the refactor’s correctness, and Junio’s earlier concerns are fully addressed.

---

### `git cat-file --batch-command` remote-object-info (v20) queued for `next`
**Headline**: Pablo Sabater’s **security-hardened series** clears its last cosmetic hurdles and is **queued for `next`** after Junio’s approval.

The series enables **efficient object metadata queries** (e.g., size) from remote repositories without downloading the full object, using the new `object-info` protocol v2 capability. The v20 iteration resolves the last open questions:
- **Dynamic capability negotiation**: The client now adapts to the server’s advertised capabilities, silently omitting unsupported placeholders (e.g., `%(objecttype)`).
- **Build fix**: Resolved a signed/unsigned comparison warning in `write_fetch_command_and_capabilities()`.
- **Documentation**: Removed redundant `CAVEATS` references in `git-cat-file.adoc` after discussion with Karthik Nayak.

**Key details**:
- **Protocol**: Full (non-abbreviated) object IDs, numeric `obj-size` values, and strict handling of unsupported object IDs.
- **Client-side command**: `remote-object-info` in `git cat-file --batch-command`, with **10,000-object batch limit** and **8KB URL length limit**.
- **Security**: Memory safety via `xcalloc`, bounds-checking, and dynamic allow-lists for metadata fields.
- **Test coverage**: 747 lines of new tests covering all three smart transports (`git://`, `file://`, `http://`), `--buffer` mode, `-Z` output, and edge cases.

**Status**: **Queued for `next`**. The series is **technically complete**, with no unresolved design or implementation concerns. The only remaining nits (translation context, alphabetical ordering) are cosmetic and will not block integration.

---

### `git rebase --update-refs` symref handling (v3) technically complete
**Headline**: Son Luong Ngoc’s **two-patch bugfix series** lands its final fixes for symref edge cases in `git rebase --update-refs`.

The series addresses scenarios where **symbolic references to branches or non-branch refs** (e.g., `main` pointing to `master`, or a symref pointing to a tag) cause rebase failures during ref updates. The v3 iteration:
- **Skips branch-to-branch symrefs** to prevent duplicate updates.
- **Safeguards non-branch symref targets** (e.g., tags) to avoid cross-worktree conflicts.
- **Deduplicates updates** when multiple aliases point to the same target.

**Key details**:
- **Files touched**: `sequencer.c`, `branch.c`, `t3404-rebase-interactive.sh`.
- **Test coverage**: Verifies HEAD aliases, secondary branch aliases, non-branch symrefs, symref chains, and cross-worktree scenarios.
- **Backend-agnostic**: Passes tests with both files and reftable backends.

**Status**: **Technically complete**. Phillip Wood’s feedback is fully addressed, and the series is ready for maintainer consideration.

---

### ODB abstraction: loose-object backend refactoring
**Headline**: Toon Claes provides **substantive reviews** on Patrick Steinhardt’s **nine-patch series** refactoring loose-object writing for ODB abstraction.

The series moves loose-object writing logic from `object-file.c` to `odb/source-loose.c`, making the loose backend fully self-contained. Toon’s reviews:
- **Patch 4/9**: Confirmed the lifted object-existence check aligns with `check_and_freshen_file()`’s documented behavior.
- **Patch 9/9**: Validated the loose backend’s self-containment, praised the removal of `the_repository`, and noted the resolution of a TODO comment from June 2026.

**Key details**:
- **Files touched**: `object-file.c`, `odb/source-loose.c`, `odb.c`, `odb.h`, and test files.
- **New symbols**: `odb_source_write_object()`, `odb_transaction_files_prepare`, `odb_transaction_files_fsync`.
- **Behavior**: No changes; this is a pure refactoring.

**Status**: **One step closer to `next`**. Toon’s reviews are **tested-level in spirit**, and the series is now awaiting broader review of the earlier patches.

---

### `git send-email` completion: usability stalemate
**Headline**: The debate over **offering both refs and patch files in completion lists** remains unresolved, with no clear path forward.

The discussion centers on whether `git send-email <TAB>` should offer:
- **Both refs and patch files** (Junio’s workflow preference, but longer lists).
- **Only refs** (SZEDER Gábor’s usability principle, with explicit filesystem completion via `./<TAB>`).

**Key details**:
- **Implementation**: `__gitcomp_file_append()` helper appends filesystem candidates to the completion list.
- **Test coverage**: Regression test verifies both patch files and refs are offered.
- **Real-world trigger**: A 14-year-old Git tag (`05c69d298c96...`) with a purely hexadecimal name.

**Status**: **Stalled**. Yury Norov’s v2 is expected to reconcile these competing concerns, but no resolution is in sight.

---

## In brief

**`git mv`**: Lucas Zamboni Orioli split the bugfix into two commits—**error message clarity** (race-free) and **early directory existence check** (TOCTOU remains). The first commit is now uncontroversial.

**`git rebase --continue --[no-]edit`**: Junio and Phillip Wood remain skeptical of the feature’s necessity, citing the existing `GIT_EDITOR=:` workaround and unresolved edge cases (e.g., `reword` steps). The patch is unlikely to proceed without addressing these concerns.

**ODB performance regression**: Patrick Steinhardt proposed a **transaction-aware ODB write redesign** to resolve the regression, but Junio emphasized the need for a **targeted fix** addressing the lookup order change in the short term. The regression may persist until late July/August.

**`git submodule` remote matching**: Éric NICOLAS’s patch to fix URL alias handling received **maintainer feedback** on commit message clarity and code polish. A v2 is expected.

**macOS regex memory leak**: Chungmin Lee’s **platform-specific workaround** fixes a severe leak in `git grep` over binary files, with detailed commit message and test coverage.

**`git-p4`**: Hardik Kumar’s patch to skip commit metadata in `extractLogMessageFromGitCommit` revealed a **historical ambiguity** in a 2007 `# fixme` comment. The thread is now blocked on clarifying the original intent.

**`git remote` push-tracking after rename (v4)**: Harald Nordgren’s series received **maintainer approval** and is ready for integration. The patch resolves URL-valued `branch.<name>.pushRemote` to a configured remote for tracking purposes.

**`git revision` `--exclude-first-parent-only`**: Junio’s **standalone bugfix** ensures the option is honored even when the first parent is already `SEEN`. Jerry Zhang’s `Reviewed-by` is recorded.

---

## On the radar

- **`git fast-import` usage refactoring**: Christian Couder’s stalled series remains in Junio’s integration tree, but its future is uncertain due to Christian’s vacation.
- **`git-p4` historical ambiguity**: The thread is blocked on clarifying the intent of a 2007 `# fixme` comment. Without resolution, the patch cannot proceed.
- **`git send-email` completion**: The usability debate remains unresolved, with no clear consensus on whether to prioritize shorter completion lists or workflow needs.
- **ODB performance regression**: A targeted fix for the lookup order change is still needed, even as Patrick Steinhardt’s transaction-aware redesign progresses.