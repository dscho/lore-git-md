# Git mailing list daily digest for 2026/08/30

## The day in brief
Domen Kožar proposed a unified `post-worktree` hook to address Junio C Hamano’s design objection against the `post-worktree-*` series, while Junio raised a long-term architectural concern about the `dk/use-nsec-runtime` series’ dependency on `repo_config_values()`. A use-after-free regression in `git stash show` was reported, and the geometric repacking race-condition fix received final maintainer approval. The `die_for_incompatible_opts()` refactoring was withdrawn, and an eight-patch series refactoring `git checkout` internals was revised and reposted.

## Notable threads

### `post-worktree-*` hooks: unified hook proposed as compromise
**What changed**: Domen Kožar proposed replacing the three separate `post-worktree-add`, `post-worktree-remove`, and `post-worktree-move` hooks with a single unified `post-worktree` hook using a subcommand-style interface (`add`, `move`, `remove`) and passing all relevant paths and the worktree ID as arguments.

**Problem or goal**: The series aims to provide reliable automation triggers for worktree lifecycle events (creation, removal, movement) to support high-velocity tooling, particularly AI-driven development environments. Junio C Hamano had raised a fundamental design objection, arguing that user-written shell wrappers could achieve the same functionality without adding Git-native hooks.

**Subsystem**: Worktree management (`builtin/worktree.c`, `worktree.c`, `worktree.h`).

**Kind of change**: Feature (new hooks).

**Technical impact**: The proposed compromise retains the same core functionality (absolute paths, worktree IDs, one event per pruned entry) but narrows the interface. Domen argues that a Git-native hook is the only reliable way to notify external tools of worktree lifecycle events triggered by callers outside the tool’s control, citing racing with other worktree operations, handling damaged entries, and the inability to require all callers to use a wrapper as reasons a wrapper-based approach is insufficient.

**Today’s development**: Domen’s proposal directly addresses Junio’s design objection by simplifying the interface while preserving observability for external tools. The email ties this to Caleb White’s earlier feedback about passing information Git already has, framing it as a way to avoid requiring hooks to query Git themselves. Domen asks whether this compromise meets Junio’s bar for a native hook, signaling openness to further iteration.

---

### CI workflow cancellation: v2 addresses edge case, but new concern raised
**What changed**: Harald Nordgren posted v2 of the CI workflow cancellation patch, addressing Junio’s edge case by including the workflow name in the concurrency group to prevent simultaneous workflows from interfering. Junio then noted that the v2 patch could cause a commit pushed to a user’s own repository (triggering a `push` event) and then submitted as a pull request (triggering a `pull_request` event) to run twice in parallel, wasting CI resources.

**Problem or goal**: Reduce wasted GitHub Actions CI runner capacity by ensuring only the latest workflow run for a pull request remains active, canceling any stale in-progress runs when a new push occurs.

**Subsystem**: CI/build system (`.github/workflows/main.yml`).

**Kind of change**: CI/build system improvement.

**Technical impact**: The v2 patch refines the concurrency control logic to cancel stale workflow runs for pull requests while preserving the existing `skip-if-redundant` optimization for regular pushes. The commit message now clarifies the old behavior (all workflows sharing a commit hash were in the same group) and the new behavior (separate groups per workflow, with cancellation enabled only for pull requests). Junio’s concern is that the same commit could end up in two separate groups (`workflow-name-commit-sha` for the push and `workflow-name-pull-request-number` for the PR), though he suspects this scenario is rare in practice.

**Today’s development**: The v2 patch is a direct response to Junio’s feedback, addressing his concern about workflow interference while maintaining the original goal. Junio’s follow-up identifies a potential regression in the `skip-if-redundant` optimization for non-PR pushes, even if the risk is low. The patch remains in review, with no objections raised.

---

### `dk/use-nsec-runtime`: maintainer raises long-term architectural concern
**What changed**: Junio C Hamano raised a long-term architectural concern about the `dk/use-nsec-runtime` series: the new dependency on `repo_config_values()` in `is_racy_stat()` forces an ordering constraint on future work, as any effort to support submodule operations in-process will require `repo_config_values()` to work on arbitrary repositories, not just `the_repository`.

**Problem or goal**: Replace the build-time `USE_NSEC` macro with a runtime configuration option (`core.useNanosec`) to allow users to enable nanosecond precision on supported filesystems without recompiling Git, reducing "racy Git" problems.

**Subsystem**: Filesystem timestamp handling (`read-cache.c`, `statinfo.c`, `environment.c`).

**Kind of change**: Feature (runtime configuration).

**Technical impact**: The series is applied under the topic `dk/use-nsec-runtime` and ready to graduate to `master`. The cross-platform correctness blocker is resolved by unconditionally including the `use_nanosec` field in `struct repo_config_values` but treating it as false when `NO_NSEC` is defined. The performance regression identified in v1 is fixed by caching the value in the repository’s `repo_config_values` struct.

**Today’s development**: Junio’s concern is forward-looking and does not block graduation. The patch itself remains ready for `master`; the concern highlights a constraint for future submodule support work. No new technical issues were raised, and the series is otherwise uncontroversial.

---

### Geometric repacking race-condition fix: maintainer seeks final confirmation
**What changed**: Junio C Hamano asked Jeff King (Peff) whether the four-patch bugfix series addressing a race condition in Git’s geometric repacking mechanism should be marked for `next`, effectively seeking a final confirmation before proceeding with integration.

**Problem or goal**: Fix a race condition where a multi-pack-index (MIDX) references a packfile removed by geometric repacking, causing processes using the stale MIDX to fail to locate objects. The issue manifests as SIGSEGV in `git replay` and missing-object errors in other operations.

**Subsystem**: Geometric repacking, MIDX, packed-object backend.

**Kind of change**: Bugfix.

**Technical impact**: The series is merge-ready and has been integrated into Junio’s `next` branch. The core fix adopts Peff’s tri-state design for `midx_fill_entry()` and gates the recovery logic on `SECOND_READ` to optimize performance for `QUICK` callers. All prior feedback has been addressed, and Peff’s final review confirmed the recovery logic in `odb/source-packed.c` is correct and optimal.

**Today’s development**: Junio’s email is a procedural step toward merging the series into `next` and, eventually, `master`. No new technical concerns were raised, and the series remains unchanged. The only remaining loose ends are minor follow-up items (error message clarity, test coverage for `git mktree --batch`, and log message demotion) that are not blockers.

---

### Negative pathspec handling: design debate continues
**What changed**: Diogo Castro pushed back against Junio C Hamano’s suggestion to recalculate the common prefix to include negative pathspecs, arguing that the "strip the common prefix" optimization is purely a performance feature for positive pathspecs, and extending it to negative pathspecs could negate its benefits. Junio then clarified that the common prefix should be recalculated to include *both* positive and negative pathspecs, but only when the negative pathspecs share a meaningful prefix with the positives.

**Problem or goal**: Fix a bug where `git ls-files` and `git add` incorrectly strip the common prefix of positive pathspecs from negative (exclusion) pathspecs, causing negative pathspecs to degenerate into empty strings or trigger out-of-bounds reads.

**Subsystem**: Pathspec handling (`dir.c`).

**Kind of change**: Bugfix.

**Technical impact**: The patch adds a guard in `do_match_pathspec()` to skip prefix-stripping for negative pathspecs. Junio’s alternative would change the semantics of negative pathspecs to be relative to the full tree rather than absolute, which could affect performance in edge cases like `git add -- a/b/c a/b/d ':!*.md'`.

**Today’s development**: The discussion remains unresolved. Diogo’s reply provides a technical rationale for preserving the performance optimization, while Junio’s clarification narrows the scope of disagreement. Both agree that the current behavior (stripping the positive-only prefix from negatives) is wrong, but they differ on whether the fix should preserve the performance optimization or prioritize intuitive semantics. No alternative implementation has been proposed yet.

---

### `git stash show` use-after-free regression reported
**What changed**: Nicolas Le Cam reported a use-after-free regression in `git stash show` when `--src-prefix` or `--dst-prefix` is used, introduced in Git 2.52.0 by commit 3ea35c64b.

**Problem or goal**: Fix a use-after-free bug causing garbage in the diff header (e.g., `diff --git Uf.txt Uf.txt` instead of `diff --git a/f.txt b/f.txt`).

**Subsystem**: Stash diff generation (`builtin/stash.c`, `diff.c`).

**Kind of change**: Bugfix.

**Technical impact**: The root cause is `OPT_STRING_F` storing pointers into `argv` elements that are later freed by `setup_revisions()`. The regression affects scriptability, as tools like `lint-staged` rely on parseable diff output. The fix will likely involve either disabling the `free_removed_argv_elements` flag for `stash show` or ensuring `diff_options` fields are copied rather than referenced.

**Today’s development**: The report is well-structured and actionable, including exact reproduction steps, expected vs. actual behavior, and a plausible root cause analysis. The regression is deterministic across versions but varies between runs in 2.53.0, strongly suggesting heap memory reuse after free. This looks like a serious regression with a narrow but real impact on scriptability.

---

## In brief
- **[2026/08/30/20-55-32 by Junio C Hamano]** Withdrew the `die_for_incompatible_opts()` refactoring series, citing dissatisfaction with the revised design and the preliminary parameter-reordering step from the first iteration. The withdrawal follows Jeff King’s substantive review expressing a preference for compile-time safety over the `EOF` sentinel adopted in v2.
- **[2026/08/30/13-42-35 by Hardik Kumar]** Fixed three minor spelling errors in comments and a test description: "fractionnal" → "fractional" in `versioncmp.c`, "occurence" → "occurrence" in `git-gui/git-gui.sh`, and "similiarity" → "similarity" in `t/t0022-crlf-rename.sh`. Michael Montalbo provided a surface-level review with "LGTM".