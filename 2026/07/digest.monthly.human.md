# The Git Project -- Monthly Digest for July 2026

**Period: 2026/07/01 -- 2026/07/31**

This month saw **exceptionally high development activity** across the Git project, with **over 2,200 emails across 543 threads** and **four major releases or release candidates** (Git 2.55.0, Git for Windows 2.55.0, Git 2.55.1, and Git 2.56.0-rc0). Traffic was **heavier than average** and **notably eventful**, with **architectural progress on multiple fronts**—ODB abstraction, reftable hardening, and `git history` subcommands all advanced significantly. The **standout developments** a reader should not miss: **Git 2.55.0 shipped** with parallel hooks, new built-ins, and the Linux fsmonitor daemon; **`git replay --linearize` was merged but immediately revealed critical post-merge issues** that are now being urgently addressed; and **the ODB alternates performance regression** sparked a fundamental architectural redesign that will shape Git’s storage layer for years to come.

---

## Key developments

### Git 2.55.0 and 2.55.1 released
**Junio C Hamano** announced the official release of **Git 2.55.0**, integrating **505 commits from 100 contributors**. This was a **major milestone release**, featuring:
- **Parallel hooks** (`hook.jobs`, `--jobs`), enabling concurrent execution of hooks defined in configuration.
- **New built-ins**: `git format-rev` (pretty-formatting revision expressions) and `git url-parse` (internal URL parsing logic).
- **Linux fsmonitor daemon**, complementing existing Windows/macOS backends.
- **Performance optimizations**: Faster revision traversal (priority queues for date-sorting), reachability bitmaps, and sparse-index operations in `git restore`.
- **Rust support**: Now enabled by default (opt-out), with preparatory work in `xdiff/` and the build system.
- **Breaking changes**: Stricter proxy URL validation (e.g., rejecting `httt://`) and disabled sideband terminal control sequences (except ANSI colors).

The release also included **over 50 bugfixes**, from memory leaks to edge cases in `git bisect` and `git describe`. **Johannes Schindelin** followed up with **Git for Windows 2.55.0**, a downstream release bundling upstream Git 2.55.0 with Windows-specific dependencies and fixes, including a critical `git fetch` regression on older Windows 10.

A **regression in `git fetch`** (introduced by the parallel-hooks work) necessitated a **Git 2.55.1 maintenance release** on July 15, containing **12 commits** that restored compatibility with older versions of Git and fixed edge cases in the new `--jobs` implementation.

**Why it matters**: This was a **feature-rich release** with significant architectural changes (ODB abstraction, Rust) and performance improvements. The volume of contributions and breadth of subsystems touched reflect active development across the project. The **parallel hooks feature** is particularly notable, as it enables **scalable CI/CD workflows** by allowing hooks to run concurrently.

---

### ODB abstraction advances, then sparks architectural redesign
The **ODB abstraction effort**, led by **Patrick Steinhardt**, saw **major progress** this month, with **four series landing in `next`**:
1. **`ps/odb-generalize-prepare`** (3 patches): Generalized the `reprepare()` callback into a more flexible `prepare()` callback with an `ODB_PREPARE_FLUSH_CACHES` flag, enabling `git grep` to work with pluggable ODB backends.
2. **`ps/odb-drop-whence`** (6 patches): Refactored `struct object_info` to replace the `whence` field with a backend-specific `struct odb_source_info`, enabling multi-source object resolution.
3. **`ps/odb-pluggable-housekeeping`** (12 patches): Refactored Git’s object housekeeping (repacking, geometric repacking, auto-gc) to be backend-agnostic, enabling future ODB backends to implement their own optimization strategies.
4. **`ps/odb-for-each-object-filter`** (9 patches): Added object filters to `odb_for_each_object()`, enabling `git cat-file --batch-all-objects` to work with pluggable backends.

However, a **12% performance regression in `git receive-pack`** (server-side pushes), introduced by the ODB abstraction rework, **sparked a fundamental architectural proposal**. The regression stemmed from a shift in object lookup order from "pack-first, loose-second" *per backend* to "pack-loose, pack-loose" *across backends*, causing redundant loose object checks in alternates directories. **Jeff King (Peff)** reframed the issue as a **systemic alternates problem**, while Patrick proposed moving alternates handling into the "files" backend itself, treating them as an implementation detail. Peff endorsed the redesign, dismissing concerns about lost flexibility as "unlikely to be practical." The proposal would enable pack-first-loose-second lookups across all alternates while simplifying commit-graph/bitmap loading. A near-term fix (targeted lookup order adjustment) may still land before Patrick’s redesign, planned for late July.

**Why it matters**: These series are **foundational steps toward pluggable ODB backends**, a long-term goal for Git’s architecture. The **alternates redesign** is particularly consequential, as it will shape how Git handles **multi-repository workflows** (e.g., partial clones, submodules) for years to come.

---

### `git replay --linearize` merged, then urgently patched
**Toon Claes**’s series adding `--linearize` to `git replay` was **merged to `master`** in early July, but **three critical issues** were identified post-merge:
1. **Silent commit dropping regression** in single-branch replay with merge commits (Johannes Schindelin, **highest priority**).
2. **CLI design inconsistency** with `git rebase` (Patrick Steinhardt, **strategic concern**).
3. **Merge commit divergence handling** (Junio Hamano, Phillip Wood, **design limitation acknowledged**).

The regression in v5 **inadvertently removed the `replayed_base` mechanism**, causing only the tip commit to be replayed while intermediate commits (including merges) are lost. **Toon Claes** posted a **v6 series** to fix the regression, restore the `replayed_base` logic, and document edge cases. The series is **ready for re-merge** and includes expanded test coverage for root commits, multiple branches, and `--contained` interactions.

However, a **philosophical debate** between **flexibility (Junio C Hamano)** and **strictness (Elijah Newren)** has stalled progress. The current implementation flattens all specified branches into a single linear history, concatenating them in an order determined by the revision walk (not the user’s command-line order). Junio argues the behavior is **intentional and reasonable**, while Elijah contends it **violates the principle of least astonishment** and creates a **recoverability asymmetry**. The outcome of this debate will set a precedent for how future commands handle **multi-branch operations**.

**Why it matters**: `git replay` is an **experimental command**, but these issues risk **data loss and user confusion**. The CLI inconsistency could limit future compatibility, and the regression fix is **urgent** for users of the feature. The **multi-branch debate** highlights a broader tension between flexibility and predictability in Git’s design.

---

### Reftable security hardening and performance fixes
**Patrick Steinhardt** posted a **12-patch series** hardening Git’s reftable backend against maliciously corrupted files. The series includes:
- **Fuzzing infrastructure** (libFuzzer support in Meson, new oss-fuzz target).
- **Test helper** (`cl_reftable_write_block`) to reduce boilerplate in unit tests.
- **Fixes for vulnerabilities** discovered by fuzzing: out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`.

The series is **fully reviewed and ready for merging**, with all feedback addressed. **Kristofer Karlsson** also contributed a **two-patch series fixing a quadratic-time tombstone handling issue** (improving runtime from ~14s to ~0.2s for 8000 refs), which is now **final-review complete** and ready for `next`.

**Why it matters**: Reftable is a **critical backend for large repositories**, and this series proactively hardens it against corruption. The **fuzzing infrastructure** is a reusable asset for future hardening work, and the **performance fix** ensures the backend scales efficiently under heavy ref churn.

---

### `git history` subcommands near completion
Two `git history` subcommands advanced significantly this month:
1. **`git history squash`**: **Harald Nordgren**’s 11-patch series implementing `git history squash` (to fold a range of commits into one while replaying descendants) reached consensus on the **commit-message template format**. The final design adopts a numbered list of commit subjects, omits `fixup!`/`amend!` messages, and retains `squash!` bodies. The series is **code-complete (v10)** and ready for final review, but **recoverability concerns** were raised about `--update-refs` operations, which cannot be undone with `git reset --hard`. Junio C Hamano **fully endorsed** the vision of using `git history squash` and `git replay` as a **faster, non-interactive backend for `git rebase --autosquash`**, calling it "an ideal future."
2. **`git history drop`**: **Patrick Steinhardt**’s 11-patch series implementing `git history drop` (to remove a commit and replay its descendants) was **accepted for merging** after resolving a **critical ref resolution dispute** in v8. The command includes conflict detection, bare repository support, and preservation of local changes. The series also modernizes the reset API, renaming `reset_head()` to `reset_working_tree()` and converting flags to an enum.

**Why it matters**: These subcommands **fill gaps in Git’s history-editing toolkit**, offering more efficient alternatives to `git rebase -i`. The `squash` feature’s `--update-refs` behavior reveals a **broader UX gap**—Git lacks a way to atomically undo complex operations.

---

### `git branch --delete-merged` lands after 23 iterations
**Harald Nordgren**’s **`git branch --delete-merged`** series, in development since May, **reached technical completion and was approved for merging** after its 23rd iteration. The command introduces **safe, automated local-branch cleanup** with comprehensive safety controls: **order-independent stacked-branch protection**, **per-branch opt-out** (`branch.<name>.deleteMerged=false`), **`--dry-run` preview**, and **repeatable upstream selectors**. The v22 revision resolved the last correctness issue (order dependency in stacked-branch protection) via a **single-pass graph traversal**, and Junio’s final approval confirmed the series is "pretty much done." The only agreed follow-up—extending stacked-branch protection to `git branch -d`—will be implemented post-merge.

**Why it matters**: This feature addresses a **long-standing pain point** for Git users: safely cleaning up merged branches without accidentally deleting work that hasn’t been integrated. The **safety mechanisms** (opt-out, dry-run, stacked-branch protection) make it suitable for **automated workflows** (e.g., CI/CD pipelines).

---

### `git cat-file --batch-command` remote-object-info graduates to `next`
**Pablo Sabater**’s **21-patch series** enabling `git cat-file --batch-command remote-object-info` was **merged to `next`** after Junio’s explicit confirmation. The feature allows Git clients to query object metadata (size, type) from remote repositories without downloading full objects, using a new `object-info` protocol v2 capability. The v21 revision addressed the last cosmetic nits (documentation, macro formatting), and the series is **structurally unchanged** from v20, which resolved all technical concerns (dynamic capability negotiation, memory safety, server-side consistency). The implementation is **robust and well-tested**, with 747 lines of new tests covering all smart transports and edge cases.

**Why it matters**: This is a **critical enabler for pluggable ODB backends**, allowing tools like `git fsck` or `git gc` to work efficiently with partial clones or alternative storage systems. The protocol’s design (e.g., omitting unsupported placeholders like `%(objectcolor)`) aligns with Git’s established patterns, and the security hardening makes it safe for production use.

---

### Memory-safety and correctness fixes
Several **memory-safety and correctness fixes** landed this month, addressing long-standing issues in Git’s codebase:
1. **`git worktree add` memory-safety bug**: **Matthias Aßhauer** posted a **critical bugfix** for `git worktree add` that prevents an out-of-bounds read leading to **silent data corruption** (recursive deletion of the working directory, including `.git`). The bug surfaces when an empty string or a string of only directory separators (e.g., `""` or `"//"`) is passed as the worktree path. Junio’s review identified a **logical flaw** in the proposed fix (potential pointer underflow), and a v2 is expected soon.
2. **`git mv` misleading error messages**: **Lucas Zamboni Orioli**’s **2-patch series** fixed misleading `git mv` errors (e.g., blaming the source file when the destination directory is missing). The series improves error messages to include both source and destination paths, adds an early existence check for the destination directory, and includes **83 new test lines** covering symlink edge cases.
3. **`git rebase --update-refs` symref bugfix**: **Son Luong Ngoc**’s **2-patch series** fixed edge cases with symbolic references in `git rebase --update-refs`. The series handles **non-branch symrefs** (e.g., tags) and **cross-worktree conflicts**, with **83 new test lines** covering symlinks as both final and intermediate components.

**Why it matters**: These fixes address **real-world pain points** for users and **critical correctness issues** in Git’s core functionality. The `git worktree add` bug, in particular, could lead to **irrecoverable data loss**, making its fix a high priority.

---

## In brief

**`git refs` subcommands merged** -- Patrick Steinhardt’s 5-patch series adding `create`, `delete`, `update`, and `rename` to `git refs` was **merged after a final typo fix**. The series consolidates reference manipulation under a unified interface.

**`excludes_file` migration into `repo_config_values`** -- Tian Yuchen’s series completed the **three-phase guardrail migration** (silent return → `BUG()` → no check) and was **merged into `next`**. The series eliminates the global `excludes_file` variable as part of the libification effort.

**`greplint.pl` test-suite linter approved** -- Michael Montalbo’s 6-patch series introducing `greplint.pl`, a linter that converts bare `grep` assertions to `test_grep`, was **approved for merging**. The linter uncovered 10+ pre-existing test bugs and is integrated into the build system.

**`GIT_TEST_LONG` CI improvements** -- Patrick Steinhardt’s 9-patch series making `GIT_TEST_LONG` tests reliable and efficient for CI is **ready for merging**. The series includes test correctness fixes, efficiency improvements, and GitLab CI parity.

**`git blame -b` usability fix** -- René Scharfe’s patch removes the extra hex digit reserved for an unused caret marker, aligning abbreviated commit hashes with `core.abbrev`. The fix was **confirmed by the original reporter** and is ready for merging.

**`git rm -n *.json` recursion bug** -- The thread about `git rm -n *.json` unexpectedly recursing into subdirectories **converged on documentation improvements**, with reviewers agreeing the behavior is correct but under-documented.

**`git replay --linearize` regression fix** -- Toon Claes’s **v6 series** to fix the silent commit-dropping regression in `git replay --linearize` is **ready for re-merge**. The series restores the `replayed_base` logic and documents edge cases.

**`git history` reorganization into `lib/`** -- Patrick Steinhardt’s RFC v3 series remains stalled due to **skepticism about the discoverability rationale**. The debate has shifted toward whether the structural benefits justify the disruption.

**`git rebase` dropped-commit notes fix** -- Phillip Wood’s 11-patch series fixing rebase-dropped-commit bugs (including notes corruption) was **merged after Junio’s final sign-off**. The series introduces `PICK_RESULT_DROPPED` to the sequencer’s result enum.

**`git merge-base` optimization** -- Tian Yuchen’s series optimizing `paint_down_to_common()` for one-sided histories is **ready for merging** after a procedural rebase. The series delivers **100-1000x speedups** for asymmetric queries.

**`git format-patch` leak fix** -- Jeff King’s 2-patch series plugs a memory leak in `git format-patch --base` and improves test suite leak reporting. The series is **ready for merging**.

**`git repo` GSoC project** -- K Jayatheerth’s weekly update (week 5) continues the research/design phase, with no on-list technical discussion or patches.

**Git Merge 2026 CFP deadline extended** -- The Call for Proposals deadline for Git Merge 2026 was extended to July 14, 2026 at 11:59 PM (UTC-7).

**`the_repository` removal efforts** -- Two series advanced:
- **Tian Yuchen’s v12** (migrating global config variables into `struct repo_config_values`) is **ready for `next`**.
- **René Scharfe’s 5-patch series** (introducing repository-aware tempfile/lockfile APIs) is **queued in `seen`**.

**`git last-modified` Bloom filter optimization** -- Toon Claes’s series aims to optimize `git last-modified` using Bloom filters but has **correctness bugs** (e.g., `--show-trees` handling) and **test gaps**.

**`git stash rename`** -- Junio C Hamano dismissed its justification as insufficient, while Brian m. Carlson provided a concrete use case. The **naming** (`rename` vs `reword`) and **behavior** remain unresolved.

**`git bisect --auto-reset`** -- Junio prefers leaving the working tree on the first bad commit (the culprit) rather than resetting to the original pre-bisect commit.

**`git repack --drop-filtered`** -- Junio identified a **logical flaw** in the `--drop-filtered` vs `--write-bitmap-index` validation. The safety mechanism is ineffective.

**`diff.<driver>.process`** -- Michael Montalbo’s **9-patch RFC** for external diff/blame tool integration **reached technical completion**. The series introduces a **long-running subprocess protocol** that lets tools inject hunks into Git’s pipelines while preserving downstream features.

**`git show-branch` slab conversion** -- Junio escalated a critique about AI-generated contributions, delaying integration.

**`wt-status` performance** -- Sahitya Chandra’s series replaces O(n²) `string_list_insert()` calls with O(n log n) append-and-sort, and was **merged**.

**`git history fixup` worktree bug** -- Toon Claes reported that `git history fixup` corrupts the index in other worktrees.

**Packfile URI race conditions** -- Ted Nyman’s v4 series fixing race conditions in Git’s HTTP transport remains **blocked by test flakiness**.

**`git send-email` filesystem path completion** -- Yury Norov’s patch to extend bash completion for `git send-email` to include filesystem paths remains **under review**.

**`git config --url` gains scp-style URL support** -- Fabian Pottbäcker’s **3-patch series** adding scp-style SSH URL support (e.g., `host:path`) to `git config --url` is **uncontroversial and ready for review**.

**UTF-8 string-width refactoring** -- Hardik Kumar’s **v2 patch** changing `utf8_strwidth()` to return `size_t` addressed reviewer concerns by updating all callers. The **overflow check removal** remains unresolved.

---

## Looking ahead

### Topics likely to dominate August
- **ODB alternates redesign**: Patrick Steinhardt’s **architectural shift** (moving alternates into the "files" backend) is the most consequential in-flight work. A near-term fix (targeted lookup order adjustment) may land before the redesign.
- **`git replay --linearize` multi-branch debate**: Toon Claes must respond to the design debate between Junio and Elijah. The series remains in `next` but is unlikely to graduate without resolution.
- **`git stash` redesign**: Junio’s suggestion to **tie stashes to branches** could subsume the `rename` feature, but no concrete proposals yet.
- **Packfile URI race conditions**: Ted Nyman’s series is **blocked by test flakiness**, but the fix is straightforward (drop the `test -s` check).
- **`git worktree add` memory-safety bug**: Matthias Aßhauer’s bugfix is **high-priority**, with a v2 expected soon.

### Ongoing efforts to watch
- **Rustification**: No updates; the effort remains **contentious** (Randall S. Becker’s NonStop port concerns vs. memory safety goals).
- **`paint_down_to_common()` optimization**: Elijah Newren’s **resumed review** of the technical document surfaced precision gaps around v1 commit-graph handling.
- **`git maintenance` gc-promisor task**: The proposed `git maintenance run --task=gc-promisor` subcommand remains under design discussion, with performance testing on large promisor packs underway.
- **`git history` signing**: Souma’s 3-patch series teaching `git history` to sign rewritten commits is **unreviewed** but well-scoped.
- **Rustification timeline**: The discussion about Git 2.55.0’s default Rust build requirement highlights the ongoing Rustification effort. The timeline for Git 3.0 (where Rust may become a requirement) remains a topic to watch, particularly for platforms like NonStop that lack Rust support.