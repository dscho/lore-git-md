# Git mailing list daily digest for 2026/08/27

## The day in brief
The `--autosquash` fix for empty commits reached a stable v4, while the trace2 hardening series faced architectural pushback over indirect `die()` calls. A new `die_for_incompatible_opts()` helper was proposed, and Junio C Hamano submitted a sweeping patch to replace `struct repository *` parameters in built-in commands with a simple `bool`. The `git checkout -m` autostash series remains blocked on performance concerns, and a French translation error was reported.

## Notable threads

### [PATCH v4] sequencer: handle empty commits with `--autosquash`
**What changed**: Farid Zakaria posted v4 of the `--autosquash` fix, addressing all prior feedback from Phillip Wood. The patch now correctly handles cases where `fixup!`/`squash!` commits cancel out their target, respecting the `--empty` option.

**Problem/goal**: Fix a long-standing edge case where Git would halt with an error instead of respecting `--empty=drop|keep|stop` when a `fixup!` or `squash!` commit empties its target.

**Technical details**:
- Introduces `is_amended_head_empty()` to detect when applying a `fixup!`/`squash!` empties the target commit
- Only the final `fixup!` in a chain may drop the target commit
- On-disk state in `$GIT_DIR/rebase-merge/fixup-target` persists across rebase restarts
- Test coverage for all three `--empty` modes and edge cases like mid-chain fixups

**Impact**: This is a bugfix for the sequencer subsystem, motivated by real-world workflows at Meta where internal LLVM patches are managed as single commits using `fixup!` autosquash. The patch is ready for final review.

---

### trace2 hardening: architectural pushback
**What changed**: Jeff King (Peff) expanded his critique of Derrick Stolee’s trace2 hardening series, arguing the current approach is a "tip of the iceberg" that misses indirect `die()` calls via helpers like `strbuf` and `json-writer.c`. Elijah Newren agreed to drop patch 3/4, replacing it with a targeted fix for `git mktree --batch`.

**Problem/goal**: Eliminate all `die()`-triggering helpers from the trace2 API to prevent Git crashes during telemetry operations.

**Technical details**:
- Peff’s review highlights that `banned-die.h` only blocks direct calls to banned functions, not transitive ones
- `strbuf` is particularly problematic because it allocates memory and bails on `snprintf()` failures
- The series now focuses on the core MIDX recovery logic (patch 4/4) and a targeted fix for `mktree --batch`

**Impact**: This is a hardening effort for the trace2 subsystem with implications for Git’s stability under memory pressure. The architectural debate centers on whether the current approach is sufficient or if a ground-up rewrite is needed.

---

### [PATCH 0/2] checkout -m: refine autostash conflict handling
**What changed**: Phillip Wood questioned the performance trade-off of the fast-path optimization in patch 2/2, suggesting it may need to be dropped or replaced with a deeper refactoring of `unpack_trees()`.

**Problem/goal**: Refine `git checkout -m` behavior when autostash is involved by making the sequencer’s autostash apply logic report conflicts and skipping the autostash retry loop when no tracked local changes exist.

**Technical details**:
- Patch 1/2 adds a `conflicted` output parameter to autostash plumbing functions
- Patch 2/2 introduces a fast-path optimization that checks for tracked changes before `unpack_trees()`
- Phillip’s review argues the pre-check overhead may outweigh the benefit of avoiding occasional redundant stash/unstash cycles

**Impact**: This is a usability improvement for the checkout subsystem, but the series is now blocked on the performance concern. The improved advice formatting in patch 2/2 remains uncontroversial.

---

### [PATCH 0/10] Make ODB fsck checks pluggable
**What changed**: Karthik Nayak reviewed the entire series, raising only minor nits and one substantive question about the use of `OPT_BIT` versus `OPT_BOOL` for the `--full` flag.

**Problem/goal**: Restructure Git’s object integrity verification (fsck) so that each ODB backend can implement its own consistency checks, preparing for future pluggable ODB backends.

**Technical details**:
- Introduces `fsck_obj_buffer()` as the sole entry point for object verification
- Backend-specific verification hooks (`odb_source_fsck_fn`)
- `struct odb_fsck_options` with `ODB_FSCK_FULL` and `ODB_FSCK_VERBOSE` flags
- Patch 4/10 fixes a long-standing discrepancy in the `--full` flag behavior

**Impact**: This is a refactoring of the fsck subsystem with no user-visible changes. The series is ready for integration pending resolution of the `OPT_BIT` question.

---

### [PATCH 0/2] branch: protect local upstreams from `git branch -d`
**What changed**: Elijah Newren identified a logical gap: the protection is not transitive, despite documentation claiming it applies to "directly or indirectly" upstream branches.

**Problem/goal**: Prevent accidental deletion of a local branch via `git branch -d` when that branch is the upstream (directly or indirectly) of another local branch.

**Technical details**:
- The implementation only checks direct dependencies, not transitive ones
- This contradicts the commit message and documentation
- The fix could involve either updating the documentation or extending the implementation to traverse the entire dependency graph

**Impact**: This is a feature addition for the branch subsystem, but the series is now blocked on the transitivity issue. Junio C Hamano has reaffirmed his backward-compatibility objection.

---

### [PATCH v2 0/3] Prevent unsafe commit operations during conflict resolution
**What changed**: Elijah Newren posted v2 addressing all prior feedback, including Phillip Wood’s maintainability concern about hardcoded paths. Junio C Hamano raised a new question about the necessity of blocking partial commits for empty commits.

**Problem/goal**: Extend Git’s existing protection against unsafe commit operations during conflict resolution to cover both `git commit --amend` and `git commit <paths>`.

**Technical details**:
- Introduces `sequencer_ongoing_operation()` helper to consolidate detection logic
- Blocks unsafe operations during conflict resolution for `git am`, `git revert`, and all `git rebase` variants
- Preserves internal use of `--amend` by interactive rebase directives (`squash`, `reword`, `edit`, `break`)

**Impact**: This is a bugfix for the merge machinery, preventing users from accidentally corrupting repository state. The series is ready for integration pending resolution of Junio’s question.

---

### [PATCH] builtin/*: stop passing the_repository to is_bare_repository()
**What changed**: Junio C Hamano rejected the patch outright, proposing a sweeping architectural change to replace the `struct repository *` parameter in built-in commands with a `bool has_repo` flag.

**Problem/goal**: Remove the implicit use of the global `the_repository` variable in built-in commands as part of the ongoing `the_repository` removal effort.

**Technical details**:
- Junio’s proposal touches 135 files, converting every built-in command’s signature
- The `bool has_repo` flag signals whether the command was invoked inside a repository
- This avoids segfaults in edge cases like `cd / && git foo -h`

**Impact**: This is a major refactoring of built-in command interfaces. The patch is blocked pending community consensus on Junio’s proposal.

---

### [PATCH 0/2] git-svn: quiet migration noise and make build optional
**What changed**: Wesley Schwengle posted v2 fixing a typo in the error message.

**Problem/goal**: Eliminate spurious migration messages and empty `.git/svn` directory creation when running `git svn info` in a plain Git repository, and make `git-svn` optional at build time.

**Technical details**:
- Patch 1/2 defers migration messages and directory creation until a legacy `refs/remotes/*` candidate is found
- Patch 2/2 introduces a `NO_GIT_SVN=YesPlease` Makefile knob
- The series is small, well-motivated, and includes test coverage

**Impact**: This is a bugfix and build system enhancement for `git-svn`, addressing long-standing annoyances.

---

### French translation error in git-checkout man page
**What changed**: Eric Gautier reported a grammatical error in the French translation of the `git-checkout` man page, where the "ne explétif" is incorrectly used.

**Problem/goal**: Correct a misleading translation that could be interpreted as "if there are *no* uncommitted changes" rather than the intended "if there *are* uncommitted changes."

**Technical details**:
- The error is in the sentence: "L’extraction échouera s’il n’y a des changements non validés..."
- The correct version should be: "L’extraction échouera s’il y a des changements non validés..."
- This is purely a documentation/translation fix with no code changes

**Impact**: This is a localization issue affecting French-speaking users. The report has been escalated to the French translation maintainers.

## In brief
- **[PATCH v13 0/4]** `checkout/switch/worktree`: improve error messages for ambiguous remote branches: Junio C Hamano marked the series as "looks good" and ready for integration.
- **[PATCH v3]** worktree: repair relative gitdir paths: Junio C Hamano accepted the functional split between `read_gitfile_raw()` and `read_gitfile_gently()`, leaving only the commit-message clarification as the remaining action.
- **[PATCH v3]** usage: reword deprecation warning: Junio C Hamano posted v3 resolving Elijah Newren’s grammatical nit.
- **[PATCH v2]** Introduce die_for_incompatible_opts() helper: Junio C Hamano posted v2 implementing the `EOF` sentinel design and fixing the `va_end()` omission.