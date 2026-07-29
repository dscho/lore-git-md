# Here is the daily digest for the Git mailing list on 2026/07/28:

---

## The day in brief

Tuesday was a **heavy but focused** day on the Git mailing list, with **78 emails across 27 threads**. The traffic was dominated by **two major themes**: (1) **bugfixes and regressions** (notably in `git rebase --update-refs`, `git cat-file --batch-command`, and the pack-bitmap subsystem), and (2) **feature refinements and design discussions** (including `git replay --linearize`, `git stash reword`, and `git add --resolved`). A security fix for Git for Windows and a build system regression tied to the Rustification effort also drew attention. The day’s tone was **constructive and forward-looking**, with several threads converging on well-defined solutions after substantive review.

The **one thing you absolutely should not miss** is the **security fix for Git for Windows** (CVE-2026-32631), which prevents credential exfiltration via symlink auto-detection. The patch is already queued for integration. For developers working on **rebase workflows**, the ongoing design discussion about `git rebase --update-refs` and symrefs is worth tracking, as it may influence future robustness improvements.

---

## Notable threads

### `git rebase --update-refs` and symref robustness
**Headline**: Design tension over whether `git rebase --update-refs` should assume HEAD always points to a branch or robustly handle symrefs to tags/other namespaces.

The thread, initiated by Son Luong Ngoc, centers on a **bug in v3 of the `git rebase --update-refs` series** that fails to handle symbolic references (symrefs) correctly. When local branches or other refs are symrefs (e.g., `main` pointing to `master`), the rebase fails even if the primary branch succeeds. Phillip Wood and Junio C Hamano engaged in a **substantive design discussion** about whether high-level commands like `rebase` should handle edge cases like HEAD pointing outside `refs/heads/` (e.g., to a tag) or simplify the logic by assuming HEAD always points to a branch.

Phillip argued that Git has historically allowed HEAD to point to any ref under `refs/` (citing commit e9cc02f0e4 from 2009), while Junio countered that user-facing commands like `git checkout` actively prevent such configurations. The discussion remains unresolved, but the **path forward is clear**: the author will likely adopt Phillip’s suggestion to use `resolved_ref` for HEAD comparison in v4, along with explanatory comments and test refinements. The latent `head_ref` buffer clobbering bug (unrelated to this patch) may become a separate follow-up.

**Status**: Under review; v4 expected. **Key files**: `sequencer.c`, `branch.c`. **Key symbols**: `refs_resolve_refdup()`, `add_decorations_to_list()`.

---

### `git replay --linearize` v8 unblocks integration
**Headline**: `--linearize` option for `git replay` now restricts to single-branch usage, resolving the "multi-branch ambiguity" that blocked the series in `next`.

Toon Claes posted **v8 of the `git replay --linearize` series**, which introduces a `--linearize` option to flatten merge commits into a linear history. The key change in v8 is a **restriction on `--linearize` usage**: it can no longer be combined with multiple revision ranges (e.g., `topic2 topic3 topic4`) or `--contained`, addressing the "multi-branch ambiguity" that had blocked the series in `next`. This aligns with Elijah Newren’s "minimal fix" proposal from v7 and eliminates emergent reachability issues while preserving the option’s core functionality.

The series is now **technically complete** and addresses all prior feedback. Junio C Hamano removed the previous version from `next` and will queue v8 for review, though he has not yet merged it to `next`. The only remaining open question is whether `--linearize` should eventually support per-branch linearization (e.g., via `--ref` syntax), but this is deferred for future work.

**Status**: Ready for review; queued for `next`. **Key files**: `replay.c`, `replay.h`, `builtin/replay.c`, `Documentation/git-replay.adoc`, `t/t3650-replay-basics.sh`. **Key symbols**: `linearize` boolean, `replay_revisions()`.

---

### `git stash reword` blocked by reflog API redesign
**Headline**: `git stash reword` requires a new reflog API (`refs_reflog_edit_in_bulk()`) to avoid data-loss risk during reflog rewrites.

Emin Özata’s **v2 of `git stash reword`** addresses all prior review feedback but is now blocked by a **fundamental correctness issue**: the current implementation slurps the entire reflog into memory, deletes it on-disk, and rewrites it, creating a critical window where the reflog exists only in memory. If the process is killed after `refs_delete_reflog()` but before the transaction commits, the reflog is permanently lost.

Junio C Hamano proposed extending the reflog API to support atomic replacement of individual entries via `refs_reflog_edit_in_bulk()`, which would:
- Accept an array of edits (replace, insert, delete) targeting specific reflog entries by index.
- Handle index drift during insert/delete operations.
- Enforce stable sorting for consistent results across backends.
- Validate impossible sequences (e.g., deleting an entry before replacing it).
- Squash multi-line messages to a single line (matching `git stash push -m` behavior).

The discussion has shifted from "should we do this?" to "how should we do this?", treating the API extension as a prerequisite for merging. The patch remains **blocked until this work is done**, but the path forward is well-defined.

**Status**: Blocked; API redesign required. **Key files**: `builtin/stash.c`, `refs.c`. **Key symbols**: `refs_reflog_edit_in_bulk()`, `ref_transaction_update_reflog`.

---

### `git cat-file --batch-command` assertion failure fixed
**Headline**: Bugfix for `git cat-file --batch-command` crash when `%(objecttype)` is omitted from the format string.

Alan Stokes reported a **crash in `git cat-file --batch-command`** when the `contents` command is issued without `%(objecttype)` in the format string. The issue, present since Git 2.43.0, was caused by a NULL dereference in `print_object_or_die()`. Jeff King (Peff) submitted a **minimal fix** that saves and restores `data->info.typep` around the `contents` command, avoiding the NULL dereference while preserving performance for `info` commands. The patch includes a new test in `t1006-cat-file.sh` and is now **queued for `next`**.

**Status**: Queued for `next`. **Key files**: `builtin/cat-file.c`, `t/t1006-cat-file.sh`. **Key symbols**: `print_object_or_die()`, `data->info.typep`.

---

### Pack-bitmap off-by-one edge case fixed
**Headline**: Bugfix for `find_objects()` in the pack-bitmap subsystem, where objects at position zero were incorrectly excluded.

David Lin posted **v2 of a bugfix** for the pack-bitmap subsystem, addressing an off-by-one edge case in `find_objects()` where objects at position zero in the base bitmap were incorrectly excluded from being recognized as already represented. This caused redundant work (reloading reachability bitmaps or adding to the not-mapped list). The fix changes the conditional check from `pos > 0` to `pos >= 0`, ensuring position zero is treated as valid. The patch includes expanded test coverage for both pseudo-merge and non-pseudo-merge scenarios and is now **accepted for integration**.

**Status**: Accepted; queued for integration. **Key files**: `pack-bitmap.c`, `t/t5333-pseudo-merge-bitmaps.sh`. **Key symbols**: `bitmap_position()`, `find_objects()`, `SEEN` flag.

---

### Security fix for Git for Windows (CVE-2026-32631)
**Headline**: Git for Windows patch prevents credential exfiltration via symlink auto-detection.

Johannes Schindelin posted a **security fix** for Git for Windows that prevents credential exfiltration when cloning a crafted repository with symlinks enabled. The vulnerability, assigned CVE-2026-32631, involves symlink type auto-detection triggering an SMB connection to a UNC path or drive-less absolute path (e.g., `\attacker\share`), exposing the user’s NTLMv2 hash. The fix modifies `compat/mingw.c` to skip auto-detection for symlink targets starting with a backslash, defaulting to file symlinks and warning users to set `symlink=dir` in gitattributes if needed. The patch is already released in Git for Windows v2.53.0(3) and is now **queued for integration**.

**Status**: Queued for integration. **Key files**: `compat/mingw.c`. **Key symbols**: `process_phantom_symlink()`.

---

### `git add --resolved` feature series
**Headline**: New `git add --resolved` option stages only conflict-resolved paths, avoiding unrelated local changes.

Junio C Hamano posted a **four-patch series** introducing `git add --resolved`, which stages only paths whose conflict markers have been removed. The feature is motivated by maintainers who frequently resolve conflicts while also having unrelated local changes in their working tree. The series includes:
1. A refactoring of conflict-marker detection logic into `is_conflict_marker_line()`.
2. A new helper, `remove_file_from_index_with_flags()`, for index removals with `--dry-run` and `--verbose` support.
3. The core `--resolved` feature in `builtin/add.c`.
4. A whitespace-only reindentation of `skip_submodule()`.

The implementation is **self-contained and well-tested**, with a new test script (`t/t2207-add-resolved.sh`) covering edge cases. The series is ready for substantive review, particularly on interaction with `--patch` and partially resolved conflicts.

**Status**: Under review. **Key files**: `builtin/add.c`, `merge-ll.c`, `read-cache.c`, `t/t2207-add-resolved.sh`. **Key symbols**: `--resolved`, `has_conflict_markers()`, `remove_file_from_index_with_flags()`.

---

## In brief

- **`git repo info` path-keys series**: K Jayatheerth’s v3 series adding path-related keys to `git repo info` is blocked by a forward-compatibility issue (use of `startup_info->prefix`, removed from `master`). The author will prepare a v4 update. **Key files**: `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1900-repo-info.sh`.
- **`git history fixup` worktree corruption**: Toon Claes and Phillip Wood discussed a bug where `git history fixup` corrupts the index in other worktrees. The fix will likely adopt `git rebase --update-refs` behavior (refuse to rewrite branches checked out in other worktrees). **Key files**: `builtin/history.c`, `worktree.c`, `read-cache.c`.
- **UTF-8 string-width documentation**: Hardik Kumar’s v5 patch documents the rationale for `utf8_strwidth()` and `utf8_strnwidth()` returning `int` (alignment with `printf()`-style APIs) and the necessity of `cast_size_t_to_int()` for overflow prevention. The patch is ready for final revision. **Key files**: `utf8.c`.
- **`git merge-base --is-ancestor` test coverage**: Nikolaus Schuetz’s test patch for `git merge-base --is-ancestor` is queued for integration. **Key files**: `t/t6010-merge-base.sh`.
- **`WITH_BREAKING_CHANGES` test failures**: Jeff King submitted a two-patch series fixing `t0014-alias.sh` failures under `WITH_BREAKING_CHANGES=1` by dynamically generating the list of deprecated commands. The series is now settled. **Key files**: `t/t0014-alias.sh`.
- **`pack.allowDuplicateObjects` RFC**: friel@openai.com posted an RFC patch introducing `pack.allowDuplicateObjects` and `--[no-]allow-duplicate-objects` for `git index-pack`. Taylor Blau raised a design question about the interaction with `--strict`/`--verify`. **Key files**: `builtin/index-pack.c`, `Documentation/git-index-pack.txt`, `t/t5308-pack-detect-duplicates.sh`.
- **`REBASE_HEAD` cleanup bug**: Long 76 and Matt Hunter reported a bug where `REBASE_HEAD` persists after a successful rebase. Phillip Wood confirmed the issue and pinpointed the root cause (cleanup logic in `sequencer.c`). **Key files**: `sequencer.c`.
- **Rustification build system fix**: Todd Zullinger proposed a fix for a build failure when `CARGO_TARGET_DIR` is set to a custom location. The fix adds `--target $*` to the `cargo build` invocation. **Key files**: `Makefile`.
- **`git merge` memory leak**: Toon Claes’s patch plugs a long-standing memory leak in `git merge` when `merge.defaultToUpstream` is enabled. The patch is queued for `seen`. **Key files**: `builtin/merge.c`, `t/t7600-merge.sh`.

---

## On the radar

- **`git stash reword` reflog API redesign**: The `refs_reflog_edit_in_bulk()` API proposed by Junio C Hamano is a significant new addition to Git’s internals. Track this for potential unification with other reflog operations (e.g., `stash drop`, `reflog expire`).
- **`git replay --linearize` per-branch support**: Toon Claes’s `--ref` proposal for per-branch linearization remains a potential future direction for the `git replay --linearize` feature.
- **ODB abstraction follow-ups**: Patrick Steinhardt’s ODB abstraction series continues to generate substantive review feedback. Track the `create_on_disk` callback design and initialization sequence discussions for future iterations.