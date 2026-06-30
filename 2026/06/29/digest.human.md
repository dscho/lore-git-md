# The Git Project Mailing List Daily Digest

**2026/06/29**

## The day in brief.

A busy Monday with 106 emails across 29 threads. The Git 2.55.0 release dominated headlines, but the day also saw significant progress on long-running architectural efforts—ODB abstraction, reftable hardening, and `git history` subcommands—alongside critical bugfixes and design debates. The most consequential news: **Git 2.55.0 shipped**, integrating 505 commits from 100 contributors, including parallel hooks, new built-ins, and the Linux fsmonitor daemon. Behind the scenes, **Patrick Steinhardt’s ODB abstraction work advanced** with multiple series landing in `next`, while **reftable security hardening** and **`git history` subcommands** neared completion. A **clock-skew regression in `git merge-base`** was fixed and merged for 2.55, and **Junio’s "What’s cooking" report** laid out the integration pipeline for the next cycle. Don’t miss the **Git for Windows 2.55.0 release** (downstream but relevant for Windows users) and the **Git Merge 2026 CFP deadline extension** (now July 14).

---

## Notable threads

### Git 2.55.0 released
**Junio C Hamano** announced the official release of Git 2.55.0, summarizing 505 non-merge commits from 100 contributors. Highlights include:
- **Parallel hooks** (`hook.jobs`, `--jobs`), enabling concurrent execution of hooks defined in configuration.
- **New built-ins**: `git format-rev` (pretty-formatting revision expressions) and `git url-parse` (internal URL parsing logic).
- **Linux fsmonitor daemon**, complementing existing Windows/macOS backends.
- **Performance optimizations**: Faster revision traversal (priority queues for date-sorting), reachability bitmaps, and sparse-index operations in `git restore`.
- **Rust support**: Now enabled by default (opt-out), with preparatory work in `xdiff/` and the build system.
- **Breaking changes**: Stricter proxy URL validation (e.g., rejecting `httt://`) and disabled sideband terminal control sequences (except ANSI colors).

The release also includes over 50 bugfixes, from memory leaks to edge cases in `git bisect` and `git describe`. **Key files touched**: `hook.c`, `builtin/format-rev.c`, `fsmonitor--daemon.c`, `reftable/`, `odb/`, and core files like `commit.c` and `revision.c`. The changelog is organized into UI/workflow, performance/internal, and fixes sections, making it easy to assess impact.

**Why it matters**: This is a major release with significant architectural changes (ODB abstraction, Rust) and performance improvements, alongside the usual bugfixes and usability tweaks. The volume of contributions and breadth of subsystems touched reflect active development across the project.

---

### Git for Windows 2.55.0 released
**Johannes Schindelin** announced Git for Windows 2.55.0, a downstream release bundling upstream Git 2.55.0 with Windows-specific dependencies (MSYS2 runtime, Git Credential Manager, OpenSSL, Bash, cURL). Notable changes:
- **Drops Windows 8.1 support** (following MSYS2).
- **Windows-specific fixes**: A `git fetch` regression causing infinite unlink loops on older Windows 10, Git Bash process shutdown issues, and parallel checkout failures with FSCache.
- **Stricter `.git` file validation**, aligning with TortoiseGit 2.19.
- **Improved handling of large clones/packfiles**.

The release includes SHA-256 checksums for all artifacts (x86_64, ARM64, 32-bit installers/archives). While not part of upstream Git, this release is critical for Windows users and reflects downstream compatibility work.

**Why it matters**: Git for Windows is the primary distribution for Windows users, and this release addresses platform-specific issues that don’t surface in upstream development. The stricter `.git` file validation is particularly relevant for interoperability with other Windows tools.

---

### ODB abstraction: `struct object_info` refactoring
**Patrick Steinhardt** posted a 6-patch series refactoring `struct object_info` to replace the `whence` field with a new `source` field, part of the ongoing ODB abstraction effort. The series enables multi-source object resolution by allowing backends to record which ODB source (loose, packed, or in-memory) an object was retrieved from.

### Key changes

- **Patch 1/6**: Adds an optional `struct odb_source_packed *source` parameter to `packed_object_info()`.
- **Patch 2/6**: Introduces `struct object_info_source` and a `sourcep` pointer in `struct object_info`.
- **Patch 3/6**: Populates the `source` field in backends (`odb/source-inmemory.c`, `odb/source-loose.c`, `packfile.c`).
- **Patches 4–5/6**: Replace `whence` usage and remove the field.
- **Patch 6/6**: Adds documentation for `struct object_info` fields.

**Design debate**: **Justin Tobler** and **Junio C Hamano** questioned whether the opt-in `source` parameter design is overcomplicating the API. The alternative—recording the ODB source once during `struct packed_git` initialization—would simplify backend logic but reduce flexibility. No consensus was reached, but Junio’s feedback suggests the current design may need revisiting.

**Why it matters**: This series is a foundational step toward pluggable ODB backends, a long-term goal for Git. The design tension between flexibility and simplicity will shape future ODB work.

---

### Reftable security hardening
**Patrick Steinhardt** posted a 12-patch series hardening Git’s reftable backend against maliciously corrupted files. The series includes:
- **Fuzzing infrastructure** (libFuzzer support in Meson, new oss-fuzz target).
- **Test helper** (`cl_reftable_write_block`) to reduce boilerplate in unit tests.
- **Fixes for vulnerabilities** discovered by fuzzing: out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`.

### Key fixes

- **Patch 3/12**: Fixes an infinite loop in `binsearch()` when the range is empty.
- **Patch 4/12**: Replaces `abort()` with `REFTABLE_FORMAT_ERROR` for invalid ref value types.
- **Patches 6–12/12**: Fix heap-buffer-overflows, out-of-bounds reads, and NULL dereferences in block parsing.

**Why it matters**: Reftable is a critical backend for large repositories, and this series proactively hardens it against corruption. The fuzzing infrastructure will help prevent regressions in this security-sensitive area.

---

### `git history drop` subcommand
**Patrick Steinhardt** posted an 11-patch series implementing `git history drop`, a new subcommand to remove a commit and replay its descendants onto its parent. The series includes:
- **Dry-run mode** (`--dry-run`) to verify the operation before execution.
- **Ref-handling**: Rejects operations with refs pointing to interior commits (with advice to use `--update-refs=head`).
- **Reset machinery refactoring**: Modernizes the reset API to avoid touching HEAD or the index unnecessarily.
- **Test coverage**: 537 lines of tests covering edge cases (bare repositories, detached HEAD, conflict detection).

**Critical bugfix**: **Junio C Hamano** identified a **logical flaw** in `find_head_tree_change()` where `--update-refs=head` could skip necessary index/worktree updates, risking repository inconsistency. The fix is pending in v8.

**Why it matters**: `git history drop` is a powerful tool for history rewriting, and the series advances the `git history` command’s capabilities. The reset machinery refactoring is also a step toward modernizing Git’s internals.

---

### Clock-skew regression in `git merge-base`
A 2-patch series from an unnamed contributor fixed a regression in `git merge-base` (without `--all`) introduced by commit 93e5b1680e (April 2025). The bug surfaced when:
1. The repository uses a v1 commit-graph (no corrected commit dates).
2. The commit topology has clock skew.
3. The correct merge base has a lower committer date than one of its ancestors.

**Fix**: Gates the early-exit optimization in `paint_down_to_common()` on whether the queue is generation-ordered. The series includes a regression test in `t6600-test-reach.sh`.

**Integration**: The series was merged for Git 2.55.0, with the in-flight `kk/merge-base-exhaustion` topic rebased on top later.

**Why it matters**: This is a critical correctness fix for a regression affecting repositories with clock skew and unregenerated commit graphs. The test coverage ensures the bug won’t resurface.

---

### `git replay --linearize` post-merge issues
**Toon Claes**’s `git replay --linearize` series was merged to `master`, but **Johannes Schindelin** and **Patrick Steinhardt** identified two critical issues:
1. **Silent commit dropping**: The base-selection logic in v5 inadvertently removed the `replayed_base` mechanism, causing only the tip commit to be replayed in single-branch scenarios.
2. **CLI design inconsistency**: `--linearize` deviates from `git rebase`’s established syntax (`--rebase-merges`, `--no-rebase-merges`).

**Follow-up work**: A patch is urgently needed to restore `replayed_base` logic or redesign multi-branch handling. The CLI inconsistency may require a design decision to align with `git rebase`.

**Why it matters**: `git replay` is an experimental command, but these issues risk data loss and user confusion. The CLI inconsistency could limit future compatibility.

---

## In brief

**Reftable memory leak fix** -- Jeff King fixed a leak in `reftable_writer_new` where the `reftable_writer` struct was allocated before input validation. The fix was merged with a trivial conflict resolution.

**`git receive-pack` hang on zero-object push** -- **Wei Hu** reported a hang in `git receive-pack` when pushing a zero-object ref update to a promisor-configured repository. **Patrick Steinhardt** confirmed the bug was already fixed by commit d9982e8290 ("connected: close err_fd in promisor fast-path") and offered to add a test case.

**`excludes_file` libification** -- **Tian Yuchen**’s series to migrate `excludes_file` into `struct repo_config_values` was blocked on a **three-phase guardrail migration** (silent return → `BUG()` → no check). **Junio C Hamano** insisted Phase 2 (adding `BUG()`) must be completed before merging.

**`greplint.pl` series** -- **Michael Montalbo**’s series to convert bare `grep` assertions to `test_grep` was resolved with a clear path to merge after targeted cleanup. The series exposed systemic risks in automated conversion masking pre-existing test bugs.

**`git history --reword` file stream leak** -- **Junio C Hamano** fixed a file stream leak and inefficient file handling in `git history --reword`. **Patrick Steinhardt** identified a missing error check on `fwrite()`, which was restored in v2.

**ODB transactions in `git-receive-pack`** -- **Justin Tobler**’s 6-patch series refactored `git-receive-pack` to use ODB transactions instead of `tmp_objdir`. The series is queued for `next` pending v2, which will address feedback on error handling and API design.

**`USE_NSEC` build-time option** -- **D. Ben Knoble**’s patch to add a `nanosec` Meson option for `USE_NSEC` sparked a debate about whether the knob should be a build-time option, runtime setting, or removed entirely. **Jeff King**’s testing showed modern Linux filesystems reliably support nanosecond timestamps, but **Brian M. Carlson** and **Patrick Steinhardt** proposed flipping the default or converting it to runtime configuration.

**GSoC 2026: Partial clone disk space recovery** -- **Siddharth Shrimali** posted a progress report on improving disk space recovery for partial clones. The work targets the promisor packfile machinery and `git maintenance`, with a proposed `git maintenance run --task=gc-promisor` subcommand.

**Git Merge 2026 CFP deadline extended** -- The Call for Proposals deadline for Git Merge 2026 was extended to July 14, 2026 at 11:59 PM (UTC-7).

---

## On the radar

**`git history squash` subcommand** -- **Harald Nordgren**’s 4-patch series implementing `git history squash` is nearing completion, with all prior feedback addressed. The series folds a commit range into its oldest commit while preserving descendant history, with a `--reedit-message` flag for commit message editing.

**`git refs` subcommands** -- **Patrick Steinhardt**’s series adding `git refs create|delete|update|rename` subcommands was merged to `next`. The series consolidates reference manipulation functionality under a unified interface.

**`kk/merge-base-exhaustion`** -- **Kristoffer Haugsbakk**’s series optimizing merge-base computation by stopping early when one side’s exclusive commits are exhausted is cooking in `next`. The series will be rebased on top of the clock-skew regression fix.

**Rustification effort** -- The Rustification effort (led by **Ezekiel Newren**) continues, with Rust now enabled by default in Git 2.55.0. The xdiff/ codebase has seen preparatory work, and the build system has been updated to support Rust.