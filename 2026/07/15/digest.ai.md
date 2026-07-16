Here is the daily digest for July 15, 2026:

---

## The day in brief

**July 15, 2026** was a **heavy but focused** day on the Git mailing list, with **153 emails across 27 threads**. The standout developments:

- **`remote-object-info` graduates to `next`**: Pablo Sabater’s GSoC series implementing `git cat-file --batch-command remote-object-info` cleared its final technical hurdle and is now queued for the next release. The feature allows clients to query object metadata (currently size) from remotes without downloading full objects, with robust security checks and dynamic capability-based validation.
- **`git history squash` completes**: Harald Nordgren’s `git history squash` feature, which folds a commit range into its oldest commit while preserving descendant history, is now functionally complete and ready for integration. The series includes an opt-out mechanism and aligns with the project’s vision of using `git history` and `git replay` as a faster, non-interactive backend for `git rebase --autosquash`.
- **ODB abstraction advances**: Patrick Steinhardt’s series introducing object filters to `odb_for_each_object()` received maintainer sign-off and is now queued for `next`. The patch enables backends to optimize traversals using reachability bitmaps, a key step toward pluggable object databases.
- **Sequencer overhaul lands**: Phillip Wood’s 9-patch series fixing how `git rebase` handles dropped commits (including external merge strategy failures and command execution failures) is now complete and approved, with only minor typofixes remaining before graduation to `next`.
- **New regressions and fixes**: A regression in `git rev-list --no-walk` path filtering was reported and diagnosed, with a fix proposed by the original author. Jeff King fixed a segfault in `git diff --relative --cached`, and Derrick Stolee addressed a crash in the trace2 subsystem caused by timestamp formatting failures.

The day’s traffic was **technically dense**, with a strong emphasis on **feature completion, refactoring, and regression fixes**. The `remote-object-info` and `git history squash` series represent significant milestones in the project’s roadmap, while the ODB abstraction and sequencer overhaul work continue to modernize Git’s internals.

---

## Notable threads

### **`remote-object-info` graduates to `next`**
**Topic**: GSoC v18 series implementing `git cat-file --batch-command remote-object-info`
**Author**: Pablo Sabater
**Status**: **Queued for `next`** after resolving the final technical hurdle (silent failure in `parse_cmd_remote_object_info()`).

The eighteenth iteration of Pablo Sabater’s security-hardened `remote-object-info` series is now complete and ready for integration. The series enables Git clients to query object metadata (currently size) from remote repositories via the `object-info` protocol v2 capability, without downloading full objects. Key features include:
- **Dynamic allow-list system**: Clients only request metadata fields the server supports, preventing segfaults from unsupported placeholders.
- **Robust security**: 10,000-object batch limit, 8KB URL length limit, integer overflow protection, and strict input validation.
- **Edge-case handling**: Graceful handling of missing objects, unsupported protocols, and malformed OIDs.
- **Test coverage**: 699 lines of new tests in `t/t1017-cat-file-remote-object-info.sh`.

The series underwent **18 revisions**, addressing feedback from Junio Hamano, Karthik Nayak, and Chandra Pratap. The final patch (11/13) resolved a silent failure mode in `parse_cmd_remote_object_info()` by replacing a `goto cleanup` with `die()`, ensuring failures are loud and unambiguous. Junio’s sign-off ("Great. Thanks all for working well together.") confirms the series is now queued for `next`.

This is a **major milestone** for the GSoC project and the broader ODB abstraction effort. The feature will enable more efficient workflows for partial clones and large repositories, and it paves the way for future expansions (e.g., object type support).

---

### **`git history squash` completes**
**Topic**: `git history squash` subcommand to fold a commit range into its oldest commit
**Author**: Harald Nordgren
**Status**: **Functionally complete (v9)** and ready for integration.

Harald Nordgren’s `git history squash` feature, which folds a commit range into its oldest commit while preserving descendant history, is now complete. The series includes:
- **Autosquash alignment**: The `--reedit-message` template now matches `git rebase -i --autosquash`, grouping `fixup!`/`squash!`/`amend!` commits under their targets.
- **Opt-out mechanism**: Users can disable the feature via `--[no-]graph-indent` or `log.graphIndent` configuration.
- **Edge-case handling**: The command now incorporates `amend!` messages targeting the first folded commit into the default squashed commit message, resolving an internal inconsistency.
- **Test coverage**: 776 lines of new tests in `t/t3455-history-squash.sh`.

The series is part of the project’s **long-term vision** to replace the internals of `git rebase -i` with a faster, non-interactive backend using `git history` and `git replay`. Junio Hamano has endorsed this goal, calling it "an ideal future" and confirming that the `squash` command is a foundational piece of this effort.

The only unresolved discussion point is whether `--reedit-message` should be the default, but this is a minor usability tweak that does not block merging. The series is now ready for Junio’s final review and integration.

---

### **ODB abstraction advances**
**Topic**: Introduce object filters to `odb_for_each_object()`
**Author**: Patrick Steinhardt
**Status**: **Queued for `next`** after maintainer sign-off.

Patrick Steinhardt’s 9-patch series introducing object filters to `odb_for_each_object()` is now complete and queued for `next`. The series enables backends to optimize object traversals using reachability bitmaps, a key step toward pluggable object databases. Key changes:
- **New interfaces**: `odb_for_each_object()` now accepts a `filter` parameter, allowing backends to skip excluded objects.
- **Bitmap optimizations**: The packed backend can now delegate iteration to `for_each_bitmapped_object()` when the filter is compatible.
- **Simplified `git cat-file`**: The final patch replaces 69 lines of backend-specific logic in `git cat-file --batch-all-objects` with 7 lines of ODB API calls.

The series received **substantive reviews** from Junio Hamano, Jeff King, Taylor Blau, and Justin Tobler, with all feedback addressed in v4. Junio’s sign-off ("Great. Thanks all for working well together.") confirms the series is now queued for `next`.

This is a **significant step forward** for the ODB abstraction effort, enabling more efficient and flexible object storage backends.

---

### **Sequencer overhaul lands**
**Topic**: Fix how `git rebase` handles dropped commits
**Author**: Phillip Wood
**Status**: **Approved (v3)** and ready for `next` after typofixes.

Phillip Wood’s 9-patch series fixing how `git rebase` handles dropped commits is now complete and approved. The series addresses:
- **External merge strategy failures**: The sequencer now reschedules picks when an external merge tool fails (e.g., due to permission errors or crashes).
- **Command execution failures**: Failed `git commit` invocations no longer cause unnecessary rescheduling.
- **Dropped commits**: Notes and post-rewrite hooks no longer receive information about commits dropped due to empty changes, fixup sequences, or skip operations.

The series introduces a new `enum pick_result` to distinguish dropped commits from successful picks, ensuring accurate error handling and resource cleanup. Junio Hamano has approved the series, and only minor typofixes remain before graduation to `next`.

This is a **substantive, tested-level contribution** to the rebase machinery, addressing long-standing technical debt and improving robustness.

---

### **Regression in `git rev-list --no-walk` path filtering**
**Topic**: Regression in `git rev-list --no-walk` path filtering (Git 2.55.0)
**Reporter**: Peter Colberg
**Status**: **Root cause identified; fix proposed**.

Peter Colberg reported a regression in Git 2.55.0 where `git rev-list --no-walk` no longer applies path-filtering arguments (e.g., `-- ':!drivers/gpu/drm/'`). The issue was introduced by commit `dd4bc01c0a8f` ("revision: use priority queue for non-limited streaming walks"), which refactored the commit-processing logic in `get_revision_1()` but omitted the `process_parents()` call that enforces path filters.

Kristofer Karlsson, the author of the problematic commit, acknowledged the regression and proposed a fix: restoring the `try_to_simplify_commit()` call (which internally invokes `process_parents()`) in the `REV_WALK_NO_WALK` case of `get_revision_1()`. The fix is narrowly scoped and aligns with the original behavior. Expect a formal patch soon, likely including tests to prevent future regressions.

---

## In brief

- **`git replay --linearize`**: Elijah Newren and Junio Hamano debated the multi-branch ambiguity with `--linearize`, with Elijah arguing for restricting the flag to a single positive ref to avoid silent concatenation of independent branches. Junio defended the current behavior as a feature, suggesting syntax alternatives to reduce ambiguity. The discussion remains unresolved, but Toon Claes is expected to respond soon.
- **`diff.<driver>.process`**: Michael Montalbo’s RFC series introducing a long-running subprocess protocol for external diff tools is now technically complete. The series enables tools to inject diff hunks into Git’s diff and blame pipelines while preserving downstream features like word diff and function context. All review feedback has been addressed, and the series is ready for maintainer sign-off.
- **`git subtree` safeguard**: Ian Jackson’s series adding a safeguard to `git subtree` to detect and reject repositories modified by a forthcoming Rust rewrite is awaiting a v2 with test style improvements. Colin Stagner confirmed the current version is correct, and Junio Hamano marked the series as "Expecting a reroll."
- **`the_repository` removal**: Patrick Steinhardt’s series removing `the_repository` from the refs subsystem and worktree code is now in `seen` and `jch`, expected to graduate to `master` soon. René Scharfe’s series introducing repository-aware tempfile and lockfile APIs is also queued for review, with Patrick Steinhardt providing substantive feedback on design choices.
- **`git fetch --submodule-errors`**: Paulius Zaleckas’s series making submodule fetch errors configurable is now technically complete. The series introduces `fetch.submoduleErrors` (config) and `--submodule-errors=(fail|warn)` (CLI) to let users choose whether `git fetch --recurse-submodules` should fail hard or merely warn when a submodule’s gitlink points to an unreachable OID. Junio Hamano requested minor mechanical improvements (array indexing type and naming) before merging.
- **`git show-branch` slab conversion**: Gatla Vishweshwar Reddy’s series converting `git show-branch` to use a commit-slab for per-commit state is now complete (v6). The series lifts the previous 27-branch limit to 64 branches and fixes behavioral regressions in `show_independent()` and `join_revs()`. Junio Hamano’s feedback has been fully addressed, and the series is ready for maintainer sign-off.
- **Coverity fixes**: Johannes Schindelin’s 11-patch series addressing unchecked return values identified by Coverity is under review. Junio Hamano identified two correctness issues (resource cleanup in `last_modified_run()` and a resource leak in `get_terms()`), which must be addressed before merging.
- **`gitweb` object ID rendering**: Travor Liu’s 2-patch series fixing object ID rendering in gitweb’s commitdiff view is under review. The series addresses a bug where index lines show full, unlinked object IDs instead of shortened, clickable hashes. The fix is narrowly scoped to `gitweb/gitweb.perl` and includes a regression test.
- **`git mv` misleading error**: Lucas Zamboni Orioli’s patch fixing a misleading error in `git mv` when the destination’s parent directory does not exist is under review. Ben Knoble raised concerns about a TOCTOU race and suggested improving the error message to include both source and destination paths.
- **Trace2 crash fix**: Derrick Stolee’s patch fixing a crash in the trace2 subsystem caused by timestamp formatting failures is under review. The patch replaces `xsnprintf()` with `snprintf()` and adds defensive initialization to avoid crashes during `atexit` events.
- **`get_commit_action()` refactoring**: Michael Montalbo’s patch refactoring `get_commit_action()` to eliminate side effects is under review. The patch makes the function a pure predicate, moving mutation logic to `simplify_commit()` to enable future lookahead optimizations.

---

## On the radar

- **`git replay --linearize`**: The multi-branch ambiguity debate remains unresolved. Toon Claes is expected to respond to Elijah Newren’s and Junio Hamano’s feedback soon.
- **`diff.<driver>.process`**: Michael Montalbo’s RFC series is ready for maintainer sign-off. Junio Hamano’s final review is pending.
- **`git subtree` safeguard**: Ian Jackson is expected to post v2 with test style improvements soon.
- **Coverity fixes**: Johannes Schindelin is expected to address Junio Hamano’s correctness concerns in a v2.
- **`git mv` misleading error**: Lucas Zamboni Orioli is expected to address Ben Knoble’s TOCTOU and error message concerns in a v2.