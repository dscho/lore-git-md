Here is the daily digest for the Git mailing list on 2026/07/10:

---

## The day in brief

A busy day on the list with **151 emails across 31 threads**, dominated by **feature finalizations, bugfixes, and integration updates**. The standout developments:

- **`git cat-file --batch-command`’s `remote-object-info` feature (Pablo Sabater) reached v16**, addressing all prior feedback and now **ready for merging**.
- **`git branch --delete-merged` (Harald Nordgren) completed its v18 iteration**, with all safety mechanisms implemented and **queued for `next`**.
- **`git history squash` (Harald Nordgren) posted v8**, functionally complete and addressing all review feedback.
- **ODB transaction series (Justin Tobler) reached v4**, converting `git-receive-pack` to use ODB transactions and **ready for `next`**.
- **Reftable performance regression fix (Kristofer Karlsson) finalized v3**, with all feedback incorporated and **ready for integration**.
- **`git replay --linearize` (Toon Claes) saw a new design challenge** from Elijah Newren about multi-branch behavior, now the **only unresolved concern**.
- **`includeIf worktree:<pattern>` (Chen Linxuan) posted v8**, unblocked and **ready for review**.
- **`js/coverity-fixes-null-safety` (Johannes Schindelin) merged to `next`**, addressing NULL-dereference vulnerabilities across 11 files.

---

## Notable threads

### `git cat-file --batch-command`’s `remote-object-info` feature reaches v16
**Topic:** `ps/cat-file-remote-object-info` (Pablo Sabater)
**Status:** **Ready for merging**

The sixteenth iteration of this security-hardened series adds `remote-object-info` support to `git cat-file --batch-command`, allowing clients to query object metadata (e.g., size) from remote repositories without downloading full objects. The series is now **technically complete**, addressing all prior feedback, including:

- **Critical refactoring flaw** in v15 (static variable inconsistency) resolved by removing the static `advertise_sid` variable entirely.
- **Memory safety improvements**: fixed a transport helper leak, loop counter hygiene, and type mismatches.
- **Dynamic capability-based validation**: runtime filtering of format placeholders against server-advertised capabilities.
- **Comprehensive test coverage**: 699 lines of new tests in `t/t1017-cat-file-remote-object-info.sh`.

The series demonstrates careful attention to security (e.g., 10,000-object batch limit, strict protocol v2 enforcement) and maintainability. All review feedback from Junio Hamano, Karthik Nayak, and others has been incorporated. The implementation is **backward-compatible** and **ready for merging**.

---

### `git branch --delete-merged` completes v18
**Topic:** Harald Nordgren’s stacked-branch protection series
**Status:** **Queued for `next`**

The v18 iteration of this series introduces `git branch --delete-merged`, a safe automated branch cleanup command with **refined stacked-branch protection**, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview. Key features:

- **Stacked-branch protection**: Fails to delete a branch if it is used as an upstream for an unmerged branch, but **clears upstream configs for kept branches** whose own upstream is being deleted.
- **Per-branch opt-out**: `branch.<name>.deleteMerged=false` exempts specific branches from automated cleanup.
- **`--dry-run` preview**: Prints "Would delete/save branch <name>" without modifying refs.

Phillip Wood’s latest review suggests **test improvements** (comments, helper functions, redundant command removal) and notes a premature `--dry-run` test case, but **no functional flaws**. The series is **implementation-complete** and **queued for `next`** pending minor test refinements.

---

### `git history squash` posts v8
**Topic:** Harald Nordgren’s `git history squash` series
**Status:** **Functionally complete**

The eighth iteration of this series introduces `git history squash`, a command to fold a commit range into its oldest commit while preserving descendant history. The v8 update addresses all prior feedback, including:

- **Autosquash alignment**: The `--reedit-message` template now **groups `fixup!`/`squash!`/`amend!` commits under their targets**, matching `git rebase -i --autosquash`.
- **Rev-list option validation**: Sanitizes all `rev-list` options (including those after `--`) by forcing `--reverse --topo-order`.
- **Merge handling**: Rejects merges with external parents but **allows fully contained merges**.

The series is **functionally complete** and **ready for integration**. The only remaining discussion point is whether `--reedit-message` should be the default, but this is a **minor usability tweak** that does not block merging.

---

### ODB transaction series reaches v4
**Topic:** `jt/receive-pack-use-odb-transactions` (Justin Tobler)
**Status:** **Ready for `next`**

The fourth iteration of this series converts `git-receive-pack` to use ODB transactions instead of `tmp_objdir`, advancing the ODB abstraction effort. Key improvements in v4:

- **Error propagation**: `odb_transaction_begin()` and `odb_transaction_commit()` now return `int` error codes.
- **Flags parameter**: Introduces `enum odb_transaction_flags` and `ODB_TRANSACTION_RECEIVE` to preserve receive-pack’s quarantine semantics.
- **Memory safety**: Fixed a latent transaction leak in `odb_transaction_commit()`.

All prior feedback from Junio Hamano and Patrick Steinhardt has been addressed. The series is **ready for `next`**, with only **deferred design questions** (e.g., merging `odb/transaction.h` and `odb/source.h`) remaining.

---

### Reftable performance regression fix finalized
**Topic:** `kk/reftable-tombstone-quadratic-fix` (Kristofer Karlsson)
**Status:** **Ready for integration**

The third iteration of this bugfix series addresses a quadratic performance regression in the reftable backend when re-creating refs that were previously deleted (tombstoned). The fix:

- Moves the `suppress_deletions` flag from the merged iterator to `struct reftable_stack_options`.
- Enables **early termination** when tombstones exceed the relevant bound.
- Improves performance from ~14s to ~0.2s in the perf test’s 8000-ref scenarios.

All feedback from Patrick Steinhardt has been incorporated, and the series is **ready for integration**. The fix is **backward-compatible** and addresses a real-world performance issue observed by brian m. carlson.

---

### `git replay --linearize` faces new design challenge
**Topic:** `tc/replay-linearize` (Toon Claes)
**Status:** **Under review**

Elijah Newren identified a **principle-of-least-astonishment violation** in the `--linearize` implementation when multiple positive refs are provided (e.g., `git replay --linearize --onto main topic1 topic2`). The current behavior flattens all commits into a single linear history, concatenating branches in an order determined by the revision walk (not the user’s command-line order). Elijah argues this is inconsistent with other `git replay` modes (`--advance` and `--revert`), which reject multiple positive refs to avoid ambiguity.

Toon Claes has not yet responded to this concern, which is now the **only unresolved issue** in the v7 series. The series is otherwise **technically complete** and **queued in `next`**.

---

### `includeIf worktree:<pattern>` posts v8
**Topic:** Chen Linxuan’s `includeIf worktree:<pattern>` series
**Status:** **Ready for review**

The eighth iteration of this series introduces `includeIf worktree:<pattern>` and `worktree/i:<pattern>` conditions to Git’s config system, enabling conditional includes based on the worktree’s path. The v8 update:

- Drops the symlink-preserving logic to resolve architectural concerns and integration conflicts.
- Reuses the `include_by_path()` helper from the first patch.
- Supports glob patterns, `~` expansion, and case-insensitive matching.

The series is now **unblocked** and **ready for review**, with all prior feedback addressed.

---

### `js/coverity-fixes-null-safety` merges to `next`
**Topic:** Johannes Schindelin’s NULL-dereference fixes
**Status:** **Merged to `next`**

This twelve-patch series addresses NULL-dereference vulnerabilities across 11 files, including:

- **`diffcore-break.c`**: Guards against NULL queue entries.
- **`remote.c`**: Uses `BUG()` for NULL `remote` in `remote_tracking()`.
- **`shallow.c`**: Fixes a latent `oid_to_hex()` bug with a stack-allocated buffer.

All feedback from Junio Hamano has been incorporated, and the series is now **cooking in `next`**.

---

## In brief

- **`git subtree` safeguard (Ian Jackson)**: v1 posted to detect and reject repositories modified by the forthcoming Rust rewrite. **Ready for v2**.
- **`git add -e` refactoring (Gatla Vishweshwar Reddy)**: v3 addresses Junio’s feedback on variable declarations and subdirectory handling. **Under review**.
- **`GIT_NO_LAZY_FETCH=fromAccepted` (Christian Couder)**: v1 introduces a security middle ground for lazy fetching. **Faces high-weight security objection** from brian m. carlson.
- **`git fetch --recurse-submodules` configurable errors (Paulius Zaleckas)**: v3 adds `fetch.submoduleErrors` and `--submodule-errors`. **Under review**.
- **`git rebase --autosquash` empty commit handling (Farid Zakaria)**: v2 addresses Phillip Wood’s feedback. **Under review**.
- **Cygwin IPv6 SCP URL fix (Ramsay Jones)**: Fixes a five-year-old regression. **Under review**.
- **`git config` `includeIf` documentation (Chen Linxuan)**: Adds `change-id` trailer to `.b4-cover-template`. **Queued for integration**.

---

## On the radar

- **`git replay --linearize` multi-branch ambiguity**: Elijah Newren’s concern about `--linearize`’s behavior with multiple positive refs remains unresolved.
- **`GIT_NO_LAZY_FETCH=fromAccepted` security objection**: brian m. carlson’s categorical objection may block the series unless addressed.
- **`git history squash` default behavior**: Whether `--reedit-message` should be the default remains an open usability question.

---