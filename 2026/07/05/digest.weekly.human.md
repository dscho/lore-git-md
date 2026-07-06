# The Git Project Mailing List Weekly Digest
**Period: 2026/06/29 -- 2026/07/05**

## The period in brief

This week saw **453 emails across 116 threads**, with traffic **heavier than average** and a **notably eventful** tone. The Git 2.55.0 release dominated early in the week, but the period was defined by **architectural progress**—ODB abstraction, reftable hardening, and `git history` subcommands all advanced significantly. The two things a reader absolutely should not miss: **Git 2.55.0 shipped** with parallel hooks, new built-ins, and the Linux fsmonitor daemon, and **`git replay --linearize` was merged but immediately revealed critical post-merge issues** that are now being urgently addressed.

---

## Key developments

### Git 2.55.0 released
**Junio C Hamano** announced the official release of Git 2.55.0, integrating **505 commits from 100 contributors**. The release is a **major milestone**, featuring:
- **Parallel hooks** (`hook.jobs`, `--jobs`), enabling concurrent execution of hooks defined in configuration.
- **New built-ins**: `git format-rev` (pretty-formatting revision expressions) and `git url-parse` (internal URL parsing logic).
- **Linux fsmonitor daemon**, complementing existing Windows/macOS backends.
- **Performance optimizations**: Faster revision traversal (priority queues for date-sorting), reachability bitmaps, and sparse-index operations in `git restore`.
- **Rust support**: Now enabled by default (opt-out), with preparatory work in `xdiff/` and the build system.
- **Breaking changes**: Stricter proxy URL validation (e.g., rejecting `httt://`) and disabled sideband terminal control sequences (except ANSI colors).

The release also includes **over 50 bugfixes**, from memory leaks to edge cases in `git bisect` and `git describe`. **Johannes Schindelin** followed up with Git for Windows 2.55.0, a downstream release bundling upstream Git 2.55.0 with Windows-specific dependencies and fixes, including a critical `git fetch` regression on older Windows 10.

**Why it matters**: This is a **feature-rich release** with significant architectural changes (ODB abstraction, Rust) and performance improvements. The volume of contributions and breadth of subsystems touched reflect active development across the project.

---

### `git replay --linearize` merged, then urgently patched
**Toon Claes**’s series adding `--linearize` to `git replay` was **merged to `master`**, but **three critical issues** were identified post-merge:
1. **Silent commit dropping regression** in single-branch replay with merge commits (Johannes Schindelin, **highest priority**).
2. **CLI design inconsistency** with `git rebase` (Patrick Steinhardt, **strategic concern**).
3. **Merge commit divergence handling** (Junio Hamano, Phillip Wood, **design limitation acknowledged**).

The regression in v5 **inadvertently removed the `replayed_base` mechanism**, causing only the tip commit to be replayed while intermediate commits (including merges) are lost. **Toon Claes** posted a **v6 series** to fix the regression, restore the `replayed_base` logic, and document edge cases. The series is **ready for re-merge** and includes expanded test coverage for root commits, multiple branches, and `--contained` interactions.

**Why it matters**: `git replay` is an **experimental command**, but these issues risk **data loss and user confusion**. The CLI inconsistency could limit future compatibility, and the regression fix is **urgent** for users of the feature.

---

### ODB abstraction advances
**Patrick Steinhardt**’s ODB abstraction work saw **significant progress**, with two series landing in `next`:
1. **`ps/odb-generalize-prepare`**: A 3-patch series generalizing the `reprepare()` callback into a more flexible `prepare()` callback with an `ODB_PREPARE_FLUSH_CACHES` flag, enabling `git grep` to work with pluggable ODB backends. The series replaces direct calls to backend-specific functions with a generic `odb_source_prepare()` and introduces `odb_prepare()` to pre-open ODB sources.
2. **`ps/odb-drop-whence`**: A 6-patch series refactoring `struct object_info` to replace the `whence` field with a backend-specific `struct odb_source_info`, enabling multi-source object resolution. The series is **conceptually approved** by Junio, with only a minor reroll needed to rename `sourcep` to `source_infop`.

**Justin Tobler** and **Junio C Hamano** questioned the opt-in `source` parameter design in `ps/odb-drop-whence`, suggesting an alternative approach that records the ODB source once during `struct packed_git` initialization. The debate remains unresolved, but the current design is **technically sound** and ready for `next`.

**Why it matters**: These series are **foundational steps toward pluggable ODB backends**, a long-term goal for Git’s architecture. The design tension between flexibility and simplicity will shape future ODB work.

---

### Reftable security hardening
**Patrick Steinhardt** posted a **12-patch series** hardening Git’s reftable backend against maliciously corrupted files. The series includes:
- **Fuzzing infrastructure** (libFuzzer support in Meson, new oss-fuzz target).
- **Test helper** (`cl_reftable_write_block`) to reduce boilerplate in unit tests.
- **Fixes for vulnerabilities** discovered by fuzzing: out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`.

The series is **fully reviewed and ready for merging**, with all feedback addressed. **Toon Claes**’s substantive review led to stricter test cases, and **Junio C Hamano** approved the test helper and 6 of the 10 fixes as "obviously correct."

**Why it matters**: Reftable is a **critical backend for large repositories**, and this series proactively hardens it against corruption. The fuzzing infrastructure is a reusable asset for future hardening work.

---

### `git history` subcommands near completion
Two `git history` subcommands advanced significantly this week:
1. **`git history drop`**: **Patrick Steinhardt**’s 11-patch series implementing `git history drop` (to remove a commit and replay its descendants) was **accepted for merging** after resolving a **critical ref resolution dispute** in v8. The command includes conflict detection, bare repository support, and preservation of local changes. The series also modernizes the reset API, renaming `reset_head()` to `reset_working_tree()` and converting flags to an enum.
2. **`git history squash`**: **Harald Nordgren**’s 4-patch series adding `git history squash` (to fold a range of commits into one while replaying descendants) reached consensus on the **commit-message template format**. The final design adopts a numbered list of commit subjects, omits `fixup!`/`amend!` messages, and retains `squash!` bodies. The series is **code-complete (v6)** and ready for final review, but **recoverability concerns** were raised about `--update-refs` operations, which cannot be undone with `git reset --hard`.

**Why it matters**: These subcommands **fill gaps in Git’s history-editing toolkit**, offering more efficient alternatives to `git rebase -i`. The `squash` feature’s `--update-refs` behavior reveals a **broader UX gap**—Git lacks a way to atomically undo complex operations.

---

### Memory-leak fixes and API design
**Jeff King** submitted a **9-patch series** plugging memory leaks in Git’s hash implementations when using non-default backends (OpenSSL SHA-256, libgcrypt). The series introduces `git_hash_discard()` and systematically applies it to leaky code paths. **Patrick Steinhardt** and **Brian M. Carlson** debated the API design, suggesting `git_hash_release()` for idiomatic consistency and idempotency for simpler error paths. The debate touches on **API design principles** and **long-term maintainability**, with implications for Git’s Rust integration.

**Why it matters**: The series is **important for OpenSSL/libgcrypt users**, and the API design debate could influence how Git designs future resource-cleanup functions.

---

## In brief

**`git cat-file --batch-command` remote object metadata support** -- Pablo Sabater’s GSoC project to add `remote-object-info` support to `git cat-file --batch-command` reached its **15th iteration** and is **ready for merging**. The series allows clients to query object metadata from remote repositories without downloading full objects.

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

---

## Looking ahead

**`git replay --linearize` regression fix** -- Toon Claes’s v6 series is **ready for re-merge** to fix the silent commit-dropping regression. Track this if you use `git replay`.

**ODB refactoring** -- Patrick Steinhardt’s `ps/odb-drop-whence` series is **ready for `next`** after a minor reroll. Critical for future ODB modularity.

**Memory-leak fixes in hash implementations** -- Jeff King’s series is **unreviewed but uncontroversial**. Important for OpenSSL/libgcrypt users.

**`git history` signing** -- Souma’s 3-patch series teaching `git history` to sign rewritten commits is **unreviewed** but well-scoped.

**Rustification timeline** -- The discussion about Git 2.55.0’s default Rust build requirement highlights the ongoing Rustification effort. The timeline for Git 3.0 (where Rust may become a requirement) remains a topic to watch, particularly for platforms like NonStop that lack Rust support.