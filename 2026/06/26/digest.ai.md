# The Git Mailing List Daily Digest for 2026/06/26

## The day in brief

A busy day on the Git mailing list (86 emails across 17 threads) saw **significant progress on several major efforts**: the `git replay --linearize` series reached v5 with all architectural concerns addressed, Patrick Steinhardt's reference backend refactoring was merged to `next`, and the `paint_down_to_common()` optimization series advanced with a critical regression identified and being fixed. **Key milestones**: the GSoC `git cat-file --batch-command` series nears completion, and Taylor Blau proposed a complex RFC to combine geometric and cruft repacking.

---

## Notable threads

### GSoC: `git cat-file --batch-command` remote object metadata (v14)
**Headline**: Final review nits addressed; series nears completion

The Google Summer of Code project to implement `git cat-file --batch-command` for remote object metadata queries saw its **fourteenth iteration** with all substantive feedback addressed. Today's traffic focused on **minor documentation and implementation nits** in the `strtoul_szt()` helper and function relocation patches.

Pablo Sabater responded to Junio Hamano's review of the `strtoul_szt()` helper, proposing a revised implementation using `uintmax_t` to handle platforms where `unsigned long` might be smaller than `size_t`. The discussion remains narrowly technical and uncontroversial. Karthik Nayak and Chandra Pratap contributed surface-level reviews on function naming consistency and commit message clarity.

**Status**: The series is **functionally complete** and addresses all prior technical feedback. Only minor documentation updates remain before merging. The implementation provides a **security-hardened** way to query object metadata (currently size) from remotes without full downloads, with dynamic format placeholder validation that adapts to server capabilities.

---

### `git replay --linearize` (v5)
**Headline**: Architectural concerns resolved; series ready for final review

Toon Claes **released v5** of the `--linearize` option for `git replay`, addressing all prior architectural concerns about the boolean refactoring and interface scope. The new version **reverts the controversial enum-to-boolean conversion** and replaces it with detailed code comments explaining the base-selection logic in `pick_regular_commit()`. A **bug in `--onto` handling** when replaying multiple divergent branches was fixed, and test coverage was expanded.

Junio Hamano reviewed the first patch, noting a minor style nit in a `BUG()` message, while the **third patch** drew attention to a **behavioral difference** from `git rebase --no-rebase-merges`: where rebase rewrites both branches of a divergent merge, replay drops one branch entirely. Junio suggested documenting this behavior and adding a test to lock in the current semantics.

**Status**: The series is **technically complete** and addresses all prior feedback. The interface design question (whether to mirror `git rebase`'s three-mode design now or later) remains open but does not block merging. Junio's "Will merge to 'next'" approval from v4 stands contingent on the resolved refactoring concerns.

---

### Reference backend refactoring: `ps/refs-onbranch-fixes` (v6)
**Headline**: Series merged to `next`; lazy-loading design approved

Patrick Steinhardt's **eleven-patch refactoring series** to resolve recursive initialization issues in Git's reference backend **was merged to `next`** under the new topic name `ps/refs-onbranch-fixes`. The series implements a **lazy-loading design** that defers write-config parsing until the first write operation, eliminating early config reads that could trigger recursion via `includeif.onbranch`.

Jeff King (Peff) provided **final architectural approval**, calling the lazy-loading approach "good" and "not too painful to maintain." The series addresses the root cause of issues where `git commit` failed to respect `onbranch`-scoped settings like `core.logAllRefUpdates`. Key changes include:
- **Deferred write-config parsing** (e.g., `core.logAllRefUpdates`, `reftable.blockSize`)
- A **recursion guard** in `get_main_ref_store()`
- **Dynamic write option passing** for the reftable backend
- Fixes for latent memory leaks in `chdir_notify` and `repo_clear()`

**Status**: **Merged to `next`** and poised to graduate to `master` unless broader testing reveals issues. This represents a **foundational shift** in how ref backends interact with config, with implications for future backend modularity.

---

### `paint_down_to_common()` optimization (v3)
**Headline**: Regression identified; series blocked pending fix

The **post-merge follow-up series** to optimize `paint_down_to_common()` for one-sided histories hit a snag when Junio C Hamano identified a **critical regression** in patch 7/8. The widened generation-monotonicity BUG assertion fires unconditionally, breaking correctness when `min_generation` is not set. The test suite caught the issue in `t6600-test-reach.sh` (test 12, `get_merge_bases_many`).

Kristofer Karlsson acknowledged the regression and plans to rework the logic for v4, either by reverting the problematic change or unifying halt conditions while preserving correctness. The rest of the series (patches 1–6/8 and 8/8) remains unaffected and ready for merging. The optimization yields **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts).

**Status**: **Blocked on regression fix** in patch 7/8. The core optimization is sound, and the test suite robustly catches edge cases. Derrick Stolee praised the trace2 instrumentation (patch 4/8) as "clean" and "over-achieve[ing]."

---

### `git history squash` (v5)
**Headline**: Edge cases and input validation debated

Harald Nordgren's `git history squash` series, which folds commit ranges into a single commit, saw **substantive review** from Phillip Wood on edge cases and input validation. The discussion focused on:
- **Single-commit ranges** (e.g., `@^!`), which are currently accepted but meaningless
- **Non-ancestor ranges** (e.g., `origin/seen^2^!` from `master`), which may confuse users
- **`fixup!`/`squash!`/`amend!` commit handling**
- **Merge commits with external parents**

Harald agreed to tighten input validation and clarify edge-case behavior in v6. Junio C Hamano weighed in on input syntax flexibility, demonstrating how `HEAD^{/pattern}` could support more complex use cases. The series is **code-complete** but may require another iteration to address these usability concerns.

**Status**: **Ready for Junio's final assessment** once edge-case handling is clarified. The core functionality (three-way merge of a range against its oldest commit's parent) is uncontested.

---

### macOS CI hangs in t5551/t5559
**Headline**: Root cause identified as Apache timeout; mitigation proposed

The long-running thread about macOS CI hangs in `t5551` and `t5559` reached a breakthrough when Patrick Steinhardt **diagnosed the root cause** as Apache's 300-second timeout being hit during `ls-refs` advertisements of 100,000 loose refs. The failure mode has shifted from HTTP/2 stalls to `curl 18` mid-transfer aborts, and similar failures are now observed in Linux CI.

The discussion converged on **increasing Apache's `Timeout` directive** as the most pragmatic solution. Michael Montalbo confirmed that a longer timeout resolves the issue in practice, and Jeff King (Peff) endorsed the approach as a "fine mitigation" that reduces failures to near-zero. No patch has been proposed yet, but the consensus is clear.

**Status**: **Mitigation strategy agreed**; patch forthcoming. The underlying Apache bug (70131) affects both HTTP/1.1 and HTTP/2, so an upstream fix would not fully resolve the issue.

---

## In brief

### **Reftable hardening (v1)**
Patrick Steinhardt's **eleven-patch series** to harden the reftable backend against corrupted files saw **surface-level review** from Christian Couder on test code hygiene. The series fixes out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage, and adds **libFuzzer-based fuzzing infrastructure** to prevent regressions.

**Status**: **Needs review**. The fuzzing infrastructure is reusable for other subsystems.

### **ODB abstraction: `ps/odb-generalize-prepare` (v1)**
Patrick Steinhardt's **two-patch series** to generalize the `reprepare()` callback into a more flexible `prepare()` callback saw **substantive review** from Toon Claes. Questions were raised about downcasting behavior and flag usage in the new API.

**Status**: **Needs review**. The refactoring enables `git grep` to work with pluggable ODB backends.

### **Promisor object connectivity checks (v3)**
Patrick Steinhardt's **five-patch series** to generalize promisor object handling in connectivity checks was **fully approved** by Christian Couder. The series replaces `find_pack_entry_one()` with `odb_for_each_object_ext()` and adds a test to verify the optimized promisor check and fallback behavior.

**Status**: **Ready for `next`**. Depends on `ps/odb-source-packed`.

### **Git 2.55.0 translation coordination**
Jiang Xin initiated the Git 2.55.0 localization cycle, noting **125 new strings** requiring translation. Junio C Hamano reminded translators that the submission deadline is June 27.

**Status**: **Ongoing**. Translators are finalizing submissions.

### **`gitk` and `git-gui` build system quieting**
Harald Nordgren's **two-patch series** to align `gitk` and `git-gui`'s translation catalog generation with core Git's quiet build conventions was **merged** for `gitk` and superseded for `git-gui` by Johannes Sixt's pre-existing work. Harald asked about integration timing.

**Status**: **Merged for `gitk`**; `git-gui` changes already present.

### **`excludes_file` migration (v2)**
Tian Yuchen's **two-patch series** to move the global `excludes_file` variable into `struct repo_config_values` saw **substantive review** from Junio C Hamano and SZEDER Gábor. Junio questioned the temporary guard (`if (repo != the_repository)`) added to avoid BUG() assertions in uninitialized submodules, while Gábor identified a **build-breaking issue** with an unused parameter in the getter function.

**Status**: **Needs reroll**. The series is part of the ongoing libification effort.

### **Rustification: loose object map**
Feng Wu's **patch** to fix a correctness issue in the Rust `ObjectMap::insert()` method was posted. The patch validates that object IDs use expected hash algorithms and returns a new `ObjectMapInsertError` enum for mismatches.

**Status**: **Needs review**. Part of the Rustification effort.

### **`git history` message prep fix**
Junio C Hamano posted a **bugfix** for the `git history` command, addressing an inefficient file-handling pattern and a file stream leak that could cause problems on Windows. The patch consolidates file handling and explicitly closes the stream before launching the external editor.

**Status**: **Needs review**. The fix is tested and merge-ready.

### **Geometric + cruft repacking (RFC)**
Taylor Blau proposed a **ten-patch RFC** to combine `git repack --geometric` and `--cruft` modes, which are currently mutually exclusive. The series introduces `--stdin-packs=follow-reachable` to `git pack-objects` and wires everything together in `git repack`. Junio C Hamano reviewed the fourth patch, suggesting a more efficient implementation for a helper function, and the eighth patch, identifying a **potential correctness issue** in the two-phase traversal logic.

**Status**: **RFC**. The series is complex and submitted during the -rc phase, so it may require significant discussion before merging.

---

## On the radar

- **`ps/odb-drop-whence` (7 commits)**: Removes the `whence` field from `struct object_info`; depends on `ps/odb-source-packed`. **Needs review**.
- **`ps/libgit-in-subdir` (3 commits)**: Moves `libgit.a` sources into a new `lib/` directory. **Needs review**.
- **`hn/branch-delete-merged` (7 commits)**: Implements `git branch --delete-merged <branch>`. **Needs review**.
- **`hn/checkout-track-fetch` (2 commits)**: Adds `--fetch` to `git checkout --track`. **Waiting for author response**.
- **`ps/shift-root-in-graph` (3 commits)**: Indents parentless root commits in `git log --graph`. **Waiting for author response**.