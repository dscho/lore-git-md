# The Git Project Daily Digest – 2026/07/08

## The day in brief.

A busy day on the list with 139 emails across 29 threads. The standout items: **Junio queued the `git replay --linearize` series** after months of review, **Patrick Steinhardt’s reftable security-hardening series cleared its final review hurdle**, and **Jeff King’s `git_hash_*()` API refactoring landed in `next`**. Smaller but notable: a **performance fix for `unpack-trees`** merged to `master`, and **Junio documented the patch-abandonment process** in `SubmittingPatches`.

---

## Notable threads

### `git replay --linearize` graduates to `next`
**Thread:** [PATCH 0/3] replay: introduce --linearize option

Toon Claes’s seven-month effort to add a `--linearize` option to `git replay` reached a milestone: **Junio marked the v7 series for `next`**, signaling it is ready for broader integration testing. The feature flattens merge commits, producing a linear history that avoids dangling commits and ensures a single topology even when multiple branches are involved. The series diverges from `git rebase --no-rebase-merges` by dropping merges and keeping only one branch, a design choice explicitly justified in the commit message.

The only unresolved edge case—combining `--linearize` and `--revert` on a partial merge range—remains a theoretical concern but is no longer blocking. The series now cooks in `next` alongside other topics, with no further changes expected unless integration testing uncovers issues.

---

### Reftable security hardening clears final review
**Thread:** [PATCH 0/12] reftable: security hardening against corrupted files

Patrick Steinhardt’s **12-patch series hardening the reftable backend against maliciously corrupted files** received its final stamp of approval from Toon Claes. The series, which began as a response to an out-of-bounds write report, evolved into a comprehensive fuzzing-driven effort that uncovered additional issues: out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`. Each fix is accompanied by a unit test, and the fuzzing infrastructure is now fully integrated into Git’s CI.

Toon’s review was substantive and tested: he manually verified that every unit test fails as expected without the corresponding code fix. The only open point is cosmetic—whether Asan output belongs in commit messages—but this is a documentation style choice, not a technical concern. The series is now **technically complete and ready for merging**, with no objections or unresolved feedback.

---

### `git_hash_*()` API refactoring lands in `next`
**Thread:** [PATCH 0/7] Make `git_hash_discard()` idempotent

Jeff King’s **seven-patch series refactoring the `git_hash_*()` API** landed in `next` after addressing all review feedback. The series makes `git_hash_discard()` idempotent (safe to call multiple times), standardizes wrapper usage via a Coccinelle semantic patch, and adds defensive runtime checks to catch misuse early. The changes are purely internal, with no user-visible behavior differences, but they harden the API against common footguns like double-finalization or premature discarding.

Patrick Steinhardt’s review of v2 was brief and positive: "The changes in this version look good to me." The series touches 26 files, with the heaviest changes in core object-writing code (`csum-file.c`, `hash.c`, `http.c`, `object-file.c`). The Coccinelle rule enforces consistent wrapper usage at compile time, while the runtime checks (added in patch 7) trigger a `BUG()` if the context is inactive. The design choice to leave `git_hash_final()` unprotected (triggering a `BUG()` on misuse) aligns with industry-standard practice for hash libraries, as explained by Brian M. Carlson.

---

### Performance fix for `unpack-trees` merged to `master`
**Thread:** [PATCH v2] unpack-trees: fix quadratic-time index scan in `next_cache_entry()`

Henrique Ferreiro’s **performance fix for `unpack-trees.c`** merged to `master`, resolving a quadratic-time index scan triggered by `git diff` with a pathspec matching a subtree whose entries appear early in the index. The fix introduces a `cache_bottom` field in `o->internal` to track forward-scan progress, allowing subsequent calls to skip already-unpacked entries. The optimization is minimal (4 lines) and preserves ordering invariants, as confirmed by Junio’s review.

Performance impact is dramatic: on a Chromium-scale repository (~500k index entries), `git diff HEAD -- .agents/OWNERS` improves from **8 minutes to 0.07 seconds**. The patch includes a perf test (`t/perf/p0009-diff-pathspec.sh`) that scales entry count via the `EXPENSIVE` prerequisite, ensuring the regression is caught early. The fix is now part of the upcoming Git release.

---

### Junio documents patch-abandonment process
**Thread:** [PATCH] SubmittingPatches: clarify how to abandon a patch series

Junio added explicit guidance to `Documentation/SubmittingPatches` on how to abandon a patch series, addressing a recurring pain point for maintainers. The new section advises contributors to send a retraction message to the list if they decide a topic is no longer viable, whether due to technical concerns, lack of time, or other reasons. The guidance includes a four-week inactivity threshold for maintainer discard and explicitly allows for resurrection later.

The patch is uncontroversial and reflects existing informal practices. Michael Montalbo provided editorial feedback (e.g., typo fixes, phrasing improvements), which Junio accepted. The update is a small but useful clarification for contributors, reducing ambiguity in the review pipeline.

---

## In brief

**Reftable tombstone performance fix** -- Patrick Steinhardt and Kristofer Karlsson resolved a quadratic performance regression in the reftable backend when re-creating refs that were previously deleted. The fix moves tombstone handling from the merged iterator to the reftable backend’s call sites, enabling early termination when tombstones exceed the relevant bound. Performance improves from ~14s to ~0.2s in the perf test’s 8000-ref scenarios. The series is under review, with a compatibility-preserving revision proposed to avoid breaking libgit2.

**ODB transaction refactoring** -- Justin Tobler’s **11-patch series converting `git-receive-pack` to use ODB transactions instead of `tmp_objdir`** advanced to v3, addressing all review feedback. The series is now **queued in Junio’s integration branches**, with no further changes expected. Key changes include making `odb_transaction_commit()` return an error code, adding environment-variable plumbing for child processes, and introducing `ODB_TRANSACTION_RECEIVE` flags to preserve `receive-pack`’s existing behavior during the transition.

**`git history reword` feedback requirements** -- The discussion around Pablo Sabater’s `git history reword` series clarified that **feedback must cover all updated refs (not just HEAD)** and that strict mode should default to on but be disableable. Patrick Steinhardt emphasized that the command intentionally allows rewriting unreachable commits, so safety checks should not break existing workflows. The thread now focuses on finalizing the output format and error messages.

**Priority queue optimization** -- Kristofer Karlsson’s **two-patch series optimizing `prio_queue_get()`** reached technical consensus, with René Scharfe providing exhaustive empirical evidence that the cascade-down approach never requires more comparisons than sift-down for extract-min operations. The series is ready for Junio’s decision on whether the modest 1-2% performance improvement justifies the code change.

**Rustification build system adjustments** -- Shardul Natu and Koji Nakamaru’s **series adding macOS Universal Binary support for Rust components** landed in `next` after addressing Junio’s feedback. The series fixes a parallel build race condition, adds CI verification for the macOS credential helper, and introduces the `RUST_TARGETS` environment variable for Universal Binary support. The unresolved artifact layout question remains deferred.

**Config lock timeout** -- Jörg Thalheim’s **config lock timeout patch** remains procedurally ready for `next`, with Johannes Schindelin endorsing the current implementation and deferring cache-related changes to a follow-up. The series introduces `core.configLockTimeout` (default 1000ms) to mitigate lock contention in concurrent Git operations.

**Bundle-URI protocol fix** -- Toon Claes and Justin Tobler’s **two-patch series fixing the bundle-URI protocol** is ready for review. The series combines client-side resilience (draining the protocol stream after invalid bundle-URI lines) with server-side validation (silently filtering empty URIs). Junio raised a philosophical question about whether the client should continue after encountering unparseable bundle-URI data, but this is not a blocker.

**`git merge --abort` autostash preservation** -- Kris Point’s **bugfix for `git merge --abort`** is blocked pending redesign. The current patch unconditionally saves the autostash even if it applies cleanly, violating intended behavior. Phillip Wood and Junio endorsed an alternative approach: decoupling reset logic from `cmd_reset()` and introducing a new flag (`RESET_WORKING_TREE_MERGE`) to control autostash behavior. The fix will target Patrick Steinhardt’s `ps/history-drop` branch in `seen`.

**HTTP daemon test helpers** -- Michael Montalbo’s **three-patch series fixing race conditions in HTTP daemon test helpers** is under review. The series replaces non-atomic shell operations (`test -f` followed by `rm` or `touch`) with atomic operations (`mv` for one-shot markers, `mkdir` for first-request election). Junio requested clarification in the commit messages for patches 1 and 2, but the fixes are otherwise uncontroversial.

---

## On the radar

**`kk/merge-base-exhaustion`** -- Kristofer Karlsson’s **10-patch series optimizing merge-base computation** remains blocked on `kk/commit-reach-find-all-fix`, which must land first or be rebased onto `master`. Junio’s guidance established a standard practice for handling patch dependencies: contributors should base their work on `master` and merge prerequisite branches into their topic branch before submission. Kristofer identified a tooling constraint (GitGitGadget’s requirement for a remote branch base) that prevented him from following this workflow, but he intends to adopt `b4` for future submissions.