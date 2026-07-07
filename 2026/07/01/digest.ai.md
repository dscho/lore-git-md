# The Git Project Mailing List Daily Digest

**2026/07/01 (Wednesday) – Busy day with major feature series and critical bugfixes**

Today’s traffic was **heavy (134 emails, 29 threads)** and **technically significant**, with several long-running feature series reaching key milestones. The standout developments:

1. **`git cat-file --batch-command` remote object metadata support** (Pablo Sabater) reached **v15** and is **ready for merging** after resolving a critical refactoring flaw in patch 5/13. The series adds `remote-object-info` support, allowing clients to query object sizes (and eventually other metadata) from remote repositories without downloading full objects. Security-hardened with dynamic capability-based validation, 680 lines of new tests, and strict protocol v2 enforcement.

2. **`git history drop`** (Patrick Steinhardt) **merged after v8** resolved a high-impact dispute over ref resolution in `find_head_tree_change()`. The series modernizes the reset API, advances `the_repository` removal, and adds a powerful new history-editing command that removes a commit and replays its descendants—with conflict detection, bare repository support, and 561 lines of test coverage.

3. **`git replay --linearize`** (Toon Claes) **converged on v6** after fixing a regression in `--linearize` behavior when replaying a single branch with merges. The series now includes an explicit justification for diverging from `git rebase`’s `--rebase-merges=<mode>` syntax, addressing Patrick Steinhardt’s requirement.

4. **`git history squash`** (Harald Nordgren) **finalized v7** with all prior feedback addressed, including template alignment with `git rebase -i` and expanded documentation. The series adds a subcommand to fold a range of commits into one, with conflict detection and 747 lines of test coverage.

5. **Coverity-driven leak fixes** (Johannes Schindelin) **queued for merging** after Junio’s explicit endorsement. The 12-patch series plugs memory, file-descriptor, and process-handle leaks across core Git, with no user-visible behavior changes.

6. **`paint_down_to_common()` optimization** (Tian Yuchen/Kristofer Karlsson) **reached v6** and is **ready for merging** after a procedural rebase. The series delivers **100-1000x speedups** for asymmetric merge-base queries by terminating early when one side’s history is exhausted.

---

## Notable threads

### `git cat-file --batch-command` remote object metadata support (Pablo Sabater)
**Headline**: **v15 ready for merging** after critical refactoring flaw fixed
**Status**: **Technically complete**; all feedback addressed, including Junio’s review of patch 5/13 (uninitialized variable + lingering global variable). The series adds `remote-object-info` support to `git cat-file --batch-command`, allowing clients to query object metadata (initially size) from remote repositories without downloading full objects. Security measures include dynamic capability-based validation, 10,000-object batch limits, and strict protocol v2 enforcement. **680 lines of new tests** in `t/t1017-cat-file-remote-object-info.sh`.
**Key improvements in v15**:
- Replaced `strtoul_szt()` with `strtoumax_szt()` for platform-independent `size_t` handling.
- Added `BUG()` calls in error paths for better debugging.
- Moved protocol documentation updates to the appropriate patch (9/13).
- **Dynamic capability-based placeholder validation** (patch 13/13) replaces static allow-lists with runtime filtering against server-advertised capabilities.
**Open question**: Error handling philosophy (silent continuation vs. explicit failure) remains unresolved but documented as non-blocking.
**Next steps**: Junio to queue for merging.

---

### `git history drop` (Patrick Steinhardt)
**Headline**: **v8 merged** after resolving critical ref resolution dispute
**Status**: **Complete**; all 11 patches merged into Junio’s integration branches. The series adds a `drop` subcommand to the experimental `git history` command, allowing users to remove a commit and replay its descendants on top of its parent. Key features:
- **Conflict detection**: Aborts if replaying descendants would result in conflicts or overwrite local changes.
- **Bare repository support**: Works in both bare and non-bare repositories.
- **Index/worktree updates**: When `HEAD` moves, the index and worktree are updated to match the new `HEAD`, preserving unrelated staged/unstaged changes.
- **Ref updates**: Moves branches pointing to the dropped commit to its parent, with configurable scope via `--update-refs=(branches|head)`.
- **Edge case handling**: Refuses to drop root commits, merge commits, or commits whose descendants contain merges (due to current limitations in `git replay`).
- **Dry-run mode**: Supports `--dry-run` to preview ref updates without modifying the repository.
- **Test coverage**: 561 lines in `t/t3454-history-drop.sh`, covering edge cases like detached `HEAD`, bare repositories, conflict detection, and preservation of local changes.
**Key refactoring**: Modernized the reset API (renamed `reset_head()` to `reset_working_tree()`, converted flags to an enum, added dry-run mode, made HEAD updates opt-in, and added an `oid_from` field for explicit "from" commit specification).
**Dispute resolution**: v8 fixed a critical flaw in `find_head_tree_change()` where the function was searching for the symbolic branch name in `result->updates[]` while `compute_pending_ref_updates()` filtered the array to contain only `HEAD` under `--update-refs=head`. The fix adds `RESOLVE_REF_READING` to `refs_resolve_ref_unsafe()` to ensure correct HEAD movement detection in detached-HEAD states.
**Next steps**: None; series is merged.

---

### `git replay --linearize` (Toon Claes)
**Headline**: **v6 converged** on interface design and regression fix
**Status**: **Technically complete**; all feedback addressed, including a regression fix for `--linearize` behavior when replaying a single branch with merges. The series adds a `--linearize` option to `git replay` that flattens merge commits, producing a linear history. Key design decisions:
- **Interface**: Uses `--linearize` as a standalone flag, diverging from `git rebase`’s `--rebase-merges=<mode>` syntax. The commit message now includes an explicit justification for this divergence, addressing Patrick Steinhardt’s requirement.
- **Behavior**: Drops merge commits and keeps only one branch of the original topology (e.g., `A->X'` or `A->Y'` for a merge of `A->X` and `A->Y`), which differs from `git rebase --no-rebase-merges` (which rewrites both branches and drops the merge, e.g., `A->X'->Y'` or `A->Y'->X'`). This difference is intentional, documented, and tested.
- **Regression fix**: Restored the `replayed_base` parameter to ensure all replayed commits are linearized into a single topology, fixing a bug where commits were dropped when replaying a single branch with merges (e.g., `master~2..master` with `--linearize --onto master~2`).
- **Test coverage**: Includes cases for replaying to the root commit, merge commit rejection, `--linearize` with `--advance`, `--revert`/`--linearize` incompatibility, replaying multiple divergent branches, and verifying the behavioral difference from `git rebase --no-rebase-merges`.
**Open question**: Whether `--linearize` should also be added to `git history` (author sees no clear use case but invites feedback).
**Next steps**: Junio to review for merging.

---

### `git history squash` (Harald Nordgren)
**Headline**: **v7 finalized** with all feedback addressed
**Status**: **Technically complete**; all prior feedback addressed, including template alignment with `git rebase -i` and expanded documentation. The series adds a `squash` subcommand to `git history` that folds a range of commits into the oldest commit in the range, replaying descendants on top. Key features:
- **Conflict detection**: Aborts if replaying descendants would result in conflicts or overwrite local changes.
- **Ref updates**: By default, refuses if any refs point to commits inside the squashed range, with an advice message suggesting `--update-refs=head` to rewrite only the current branch.
- **Edge case handling**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **`--reedit-message`**: Seeds the editor with a template identical to `git rebase -i`’s squash behavior, addressing prior usability concerns about template clutter. The template includes:
  - An "N commits" banner.
  - Each folded message under its own header, following autosquash rules:
    - `fixup!` messages: fully commented out under a "will be skipped" header.
    - `squash!`/`amend!` messages: bodies retained with only the marker subject commented (allowing rewording of remarks).
- **Test coverage**: 747 lines in `t/t3455-history-squash.sh`, covering range parsing, merge commit handling, ref safety, authorship preservation, `--reedit-message` template construction, and `--update-refs` behavior.
**Key refactoring**: Extracted sequencer helpers (`add_squash_combination_header`, `add_squash_message_header`, `squash_subject_comment_len`) to `sequencer.h` for reuse by `git history squash`, ensuring template consistency.
**Open questions**:
- Whether `--reedit-message` (or `--edit`) should be the default (Phillip Wood advocates for this as a commit hygiene measure).
- Short-term mitigation for recoverability concerns (e.g., improved documentation or a `--dry-run --verbose` mode listing affected refs).
**Next steps**: Junio to review for merging.

---

### Coverity-driven leak fixes (Johannes Schindelin)
**Headline**: **12-patch series queued for merging** after Junio’s endorsement
**Status**: **Technically complete**; all patches carry Reviewed-by from Patrick Steinhardt, Jeff King, Eric Sunshine, and/or Junio C Hamano. The series plugs memory, file-descriptor, and process-handle leaks across core Git, with no user-visible behavior changes. Key fixes:
- **Patch 1/12 v2**: Ensures `FILE *fp` is closed on all error paths in `load_one_loose_object_map()` by consolidating cleanup into a single `err:` exit path. Replaced unsafe `errno`-based error detection with `ret = ferror(fp) ? -1 : 0` to reliably distinguish between EOF and I/O errors.
- **Patch 2/12 v2**: Adds a guard (`if (fd >= 0) close(fd)`) in `write_one_object()` to prevent undefined behavior when `open()` fails.
- **Patch 3/12**: Plugs a file-descriptor leak in `download_https_uri_to_file()` by ensuring the `child_out` file handle is closed when `finish_command(&cp)` fails.
- **Patch 4/12 v2**: Replaces `if (cmd->in)` / `if (cmd->out)` with `if (cmd->in > 0)` / `if (cmd->out > 0)` in `start_command()`’s error-path rollback to prevent `close(-1)`.
- **Patch 5/12 v2**: Removes a redundant intermediate copy of `struct line_log_data` in `line_log_process_ranges_arbitrary_commit()` that was leaking whenever `bloom_filter_check()` determined a commit did not touch any tracked paths.
- **Patch 6/12**: Plugs two memory leaks in `read_one_dir()` by adding explicit `free(ud.untracked)` and `free(ud.dirs)` calls on early-return paths.
- **Patch 7/12**: Plugs a memory leak in `get_superproject_working_tree()` by redirecting two early-return paths through a new `out:` label that frees `cwd`, `sb.buf`, and `one_up.buf`.
- **Patch 8/12 v2**: Consolidates all cleanup in `get_superproject_working_tree()` into a single exit path (`out:` label) that releases `cwd`, `sb.buf`, and `one_up.buf`, eliminating fragility and plugging a latent leak of `one_up`’s backing buffer.
- **Patch 9/12**: Plugs four memory leaks in `builtin/worktree.c` by redirecting an early error return through the existing `cleanup:` label so that `path`, `opt_track`, `branch_to_free`, and `new_branch_to_free` are freed.
- **Patch 10/12**: Plugs a memory leak in `curl_append_msgs_to_imap()` by adding `strbuf_release(&msgbuf.buf)` alongside existing libcurl cleanup calls.
- **Patch 11/12**: Plugs a memory leak in `reftable_table_refs_for_unindexed()` by extending the `out:` cleanup block to release `filter->oid` and free `filter`.
- **Patch 12/12**: Plugs a handle leak and latent use-after-free in Windows-specific process termination logic by enforcing clear ownership: `exit_process()` and `terminate_process_tree()` now unconditionally own and close the process handle passed to them, removing the redundant `CloseHandle(h)` call in `mingw_kill()` that Coverity flagged as a use-after-free (CID 1437238).
**Open question**: An orthogonal bug in `read_one_dir()` (uninitialized slots in `->dirs` and `->untracked` on failure paths) may cause `free_untracked()` to dereference garbage pointers, risking a crash. Johannes Schindelin plans to address this separately.
**Next steps**: Junio to queue for merging.

---

### `paint_down_to_common()` optimization (Tian Yuchen/Kristofer Karlsson)
**Headline**: **v6 ready for merging** after procedural rebase
**Status**: **Technically complete**; all feedback addressed, including a procedural rebase on `kk/commit-reach-find-all-fix` to resolve conflicts and test failures. The series optimizes `paint_down_to_common()` to terminate early when one side of a merge-base query exhausts its commit queue, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts or shallow histories). Key improvements:
- **Performance**: Delivers **100-1000x speedups** for asymmetric queries. For example, `merge-base --all v2.0.0 v2.55.0-rc1` on the Linux kernel repo drops from **3.67s to 17ms** (v1 commit-graph) with **5,725 steps** (down from 167,413).
- **Algorithm**: Introduces a `paint_state` struct to track per-side commit counters (`parent1_count`, `parent2_count`, `mb_candidate_count`) and replaces the old `nonstale_queue` termination logic with a counter-based approach. The loop now exits when counters reach zero, which happens one iteration earlier than the old `max_nonstale`-based termination in some topologies.
- **Documentation**: Adds `Documentation/technical/paint-down-to-common.adoc`, explaining the algorithm, use cases, generation-number regions, termination conditions, side-exhaustion logic, and trace2 instrumentation.
- **Test coverage**: Includes edge cases (self/duplicate/pending-stale/infinity-both-sides, diamond topologies, mixed finite/INFINITY generation numbers, clock-skew topologies), deterministic step-count assertions for all commit-graph modes, and regression tests for the fallback scenario.
- **Cleanup**: Removes the commit-date ordering fallback entirely (patch 10/10), as the side-exhaustion optimization renders it obsolete.
**Key patches**:
- **Patch 1/10**: Adds technical documentation.
- **Patch 2/10**: Adds `test_trace2_data_singular()` to improve diagnostic output for trace2 assertions.
- **Patch 6/10**: Introduces clock-skew topologies to test side-exhaustion edge cases with v1 commit graphs.
- **Patch 7/10**: Introduces the `paint_state` struct and replaces the old `nonstale_queue` termination logic.
- **Patch 8/10**: Implements the core early-termination logic, delivering the **100-1000x speedups**.
- **Patch 10/10**: Removes the commit-date ordering fallback entirely.
**Next steps**: Junio to queue for merging.

---

## In brief

### `git blame -b` output formatting (René Scharfe)
**Topic**: **Bugfix for commit hash abbreviation mismatch**
**Status**: **Patch confirmed by original reporter**; ready for merging.
**Details**: Fixes a usability friction where `git blame -b` reserved an extra hex digit for a caret marker that was never displayed, causing non-boundary commit hashes to exceed the user’s `core.abbrev` setting by one character. The patch refactors the mark-handling logic in `blame.c` to count and print boundary/ignored/unblamable marks only when they are actually shown, ensuring the abbreviated commit hash length aligns with `core.abbrev`. **Tested-by** from the original reporter (Laszlo Ersek).
**Files touched**: `blame.c`, `Documentation/git-blame.adoc`, `t/t8002-blame.sh`.
**Next steps**: Junio to merge.

---

### `git history` subcommand refactoring (Patrick Steinhardt)
**Topic**: **Extract function to drop unmerged entries to stage 0**
**Status**: **Patch 1/11 of `git history drop` series merged**.
**Details**: Refactors `repo_read_index_unmerged()` to extract the logic that drops unmerged index entries to stage 0 into a new reusable function `index_state_unmerged_to_stage0(struct index_state *)`. The patch also fixes a signedness mismatch in the loop counter (`int i` -> `unsigned int i`). Pure refactoring with no behavior change.
**Files touched**: `read-cache.c`.
**Next steps**: None; patch is merged.

---

### Rustification build system adjustment (Shardul Natu/Koji Nakamaru)
**Topic**: **macOS Universal Binary support for Rust components**
**Status**: **Patch ready for integration**.
**Details**: Adds support for macOS Universal Binaries by allowing users to specify multiple Rust target triples (e.g., `aarch64-apple-darwin x86_64-apple-darwin`) via the `RUST_TARGETS` environment variable. The build system compiles separate static libraries per target and combines them into a single Universal Binary using `lipo`. Also ensures the `git-credential-osxkeychain` helper is properly linked with Rust libraries when Rust support is enabled.
**Files touched**: `Makefile`.
**Next steps**: Junio to merge.

---

### HTTP/HTTPS authentication regression (Git for Windows)
**Topic**: **NTLM authentication broken in Git for Windows 2.55.0.windows.1**
**Status**: **Redirected to Git for Windows issue tracker**.
**Details**: A user reported that HTTP/HTTPS authentication using domain account passwords (NTLM) broke after upgrading Git for Windows from 2.53.0.windows.2 to 2.55.0.windows.1. The previously effective workaround `http."<url>".allowNTLMAuth true` no longer restores functionality. Johannes Schindelin redirected the discussion to the Git for Windows issue tracker ([#6308](https://github.com/git-for-windows/git/issues/6308)).
**Next steps**: Follow up in Git for Windows issue tracker.

---

### `git gui` encoding mismatch (Martin Malec)
**Topic**: **Fix for non-ASCII home directories on Windows**
**Status**: **Patch proposed**.
**Details**: Fixes a bug where `git gui` fails to start when the user’s home directory contains non-ASCII characters (e.g., accented letters from EntraID/AzureAD profiles). The root cause is an encoding mismatch: `git-gui.tcl` uses Tcl’s default system encoding (ANSI code page) to capture the output of `cygpath -m`, which emits UTF-8. The fix replaces the `safe_exec` call with `safe_open_command` and explicitly sets `-encoding utf-8` to match `cygpath`’s output.
**Files touched**: `git-gui.tcl`.
**Next steps**: Review by Git for Windows team.

---

### Meson build race (D. Ben Knoble)
**Topic**: **Fix build race in generating hook-list.h**
**Status**: **Patch ready for merging**.
**Details**: Restores `hook-list.h` to the `builtin_sources` list in `meson.build` so the header is generated before `builtin/bugreport.c` is compiled, eliminating a race condition that causes build failures when the header is missing. The issue was introduced by commit `2eb541e8f2a9` ("hook: move is_known_hook() to hook.c for wider use").
**Files touched**: `meson.build`.
**Next steps**: Junio to merge.

---

## On the radar

### `git repo` command (K Jayatheerth)
**Topic**: **GSoC project update**
**Status**: **Still in research/design phase**; no patches or technical discussion on-list.
**Details**: The project to develop a new `git repo` command remains in its early stages, with weekly progress communicated via external blog posts. No technical specifications, design documents, or patches have been shared on the mailing list. The mailing list serves as a notification channel for blog updates.
**Next steps**: Watch for transition to patch submission and on-list technical discussion.

---

### `git send-email` improvements (Aditya Garg)
**Topic**: **SMTP server configuration and IMAP folder integration**
**Status**: **No new traffic today**; last update in June.
**Details**: Aditya Garg’s work on `git send-email` improvements (SMTP server configuration options, IMAP folder integration for archiving sent emails) has not generated new traffic. The series is likely still under review or awaiting reroll.
**Next steps**: Monitor for new iterations or review feedback.

---

### `git hook` configuration (Adrian Ratiu)
**Topic**: **Hook subsystem improvements**
**Status**: **No new traffic today**; last update in June.
**Details**: Adrian Ratiu’s effort to enable hooks to be specified and configured via Git configuration (rather than fixed filesystem paths) has not generated new traffic. The series includes a `git hook list` subcommand and related plumbing to support multiple hooks per event.
**Next steps**: Monitor for new iterations or review feedback.

---

### `git maintenance` improvements (Karthik Nayak)
**Topic**: **Ref backend and fetch/push plumbing improvements**
**Status**: **No new traffic today**; last update in June.
**Details**: Karthik Nayak’s work on `git maintenance` infrastructure, ref backends (reftable), and fetch/push plumbing improvements has not generated new traffic. The series is likely still under review or awaiting reroll.
**Next steps**: Monitor for new iterations or review feedback.

---

### `git fsmonitor` improvements (Paul Tarjan)
**Topic**: **Linux fsmonitor implementation**
**Status**: **No new traffic today**; last update in June.
**Details**: Paul Tarjan’s work on the Linux fsmonitor implementation (inotify-based daemon) has not generated new traffic. The series brought Linux to parity with the existing Windows and macOS backends.
**Next steps**: Monitor for new iterations or review feedback.

---

### `git sparse-checkout` / `git sparse-index` (Derrick Stolee/Elijah Newren)
**Topic**: **Sparse-checkout and sparse-index improvements**
**Status**: **No new traffic today**; last update in June.
**Details**: Derrick Stolee and Elijah Newren’s work on sparse-checkout and sparse-index (to make Git perform well in monorepos where only a subset of files is checked out) has not generated new traffic. The sparse-index avoids expanding the full index for operations that do not need it.
**Next steps**: Monitor for new iterations or review feedback.