# The Git Project Mailing List Weekly Digest
**Period: 2026/06/29 -- 2026/07/05**

---

## The period in brief

This week (2026/06/29--2026/07/05) was **exceptionally active and milestone-heavy**, with **553 emails across 125 threads** spanning six days. Traffic was **heavier than usual** (averaging 92 emails/day), driven by the **Git 2.55.0 release**, **major architectural advancements**, and **critical post-merge bugfixes**. The tone was **collaborative and forward-looking**, with several long-running efforts reaching resolution. **Three developments you absolutely should not miss**:
- **Git 2.55.0 shipped**, integrating 505 commits including parallel hooks, new built-ins, and the Linux fsmonitor daemon.
- **Patrick Steinhardt’s ODB abstraction work advanced significantly**, with multiple series landing in `next`.
- **`git replay --linearize` and `git history drop` were merged**, but both required urgent post-merge fixes for critical regressions.

---

## Key developments

### Git 2.55.0 released
**Junio C Hamano** announced the official release of Git 2.55.0, summarizing 505 non-merge commits from 100 contributors. This is a **major release** with significant architectural changes and performance improvements. Highlights include:
- **Parallel hooks** (`hook.jobs`, `--jobs`), enabling concurrent execution of hooks defined in configuration.
- **New built-ins**: `git format-rev` (pretty-formatting revision expressions) and `git url-parse` (internal URL parsing logic).
- **Linux fsmonitor daemon**, complementing existing Windows/macOS backends.
- **Performance optimizations**: Faster revision traversal (priority queues for date-sorting), reachability bitmaps, and sparse-index operations in `git restore`.
- **Rust support**: Now enabled by default (opt-out), with preparatory work in `xdiff/` and the build system.
- **Breaking changes**: Stricter proxy URL validation (e.g., rejecting `httt://`) and disabled sideband terminal control sequences (except ANSI colors).

The release also includes over 50 bugfixes, from memory leaks to edge cases in `git bisect` and `git describe`. **Key files touched**: `hook.c`, `builtin/format-rev.c`, `fsmonitor--daemon.c`, `reftable/`, `odb/`, and core files like `commit.c` and `revision.c`.

**Why it matters**: This release marks a **significant step forward** in Git’s architecture, with ODB abstraction, Rust integration, and performance improvements setting the stage for future work. The volume of contributions and breadth of subsystems touched reflect active development across the project.

---

### ODB abstraction advances
**Patrick Steinhardt**’s ODB abstraction work saw **major progress** this week, with two series landing in `next` and a third converging on design. The effort aims to enable pluggable object database backends, a long-term goal for Git’s architecture.

#### `struct object_info` refactoring
A 6-patch series refactoring `struct object_info` to replace the `whence` field with a new `source` field reached **final approval** and was queued for merging. The series enables multi-source object resolution by allowing backends to record which ODB source (loose, packed, or in-memory) an object was retrieved from. **Key changes**:
- Adds an optional `struct odb_source_packed *source` parameter to `packed_object_info()`.
- Introduces `struct object_info_source` and a `sourcep` pointer in `struct object_info`.
- Populates the `source` field in backends (`odb/source-inmemory.c`, `odb/source-loose.c`, `packfile.c`).
- Replaces `whence` usage and removes the field.

**Design debate**: **Justin Tobler** and **Junio C Hamano** questioned whether the opt-in `source` parameter design is overcomplicating the API. The alternative—recording the ODB source once during `struct packed_git` initialization—would simplify backend logic but reduce flexibility. No consensus was reached, but Junio’s feedback suggests the current design may need revisiting.

**Why it matters**: This series is a **foundational step** toward pluggable ODB backends. The design tension between flexibility and simplicity will shape future ODB work.

#### ODB `prepare()` callback generalization
A 3-patch series generalizing the `reprepare()` callback into a more flexible `prepare()` callback with an `ODB_PREPARE_FLUSH_CACHES` flag was **approved for `next`**. The series enables `git grep` to work with pluggable ODB backends by replacing direct calls to backend-specific functions (e.g., `odb_source_packed_prepare()`) with a generic `odb_source_prepare()` that delegates through the vtable. **Key changes**:
- New `enum odb_prepare_flags` with `ODB_PREPARE_FLUSH_CACHES`.
- `odb_transaction_begin()` now takes flags and disallows `NULL` output on success.
- Backward compatibility preserved via `odb_reprepare()` wrapper.

**Why it matters**: This is a **critical step** toward pluggable ODB backends, enabling `git grep` and other commands to work with alternative backends.

---

### Reftable security hardening
**Patrick Steinhardt** posted a 12-patch series hardening Git’s reftable backend against maliciously corrupted files. The series includes:
- **Fuzzing infrastructure** (libFuzzer support in Meson, new oss-fuzz target).
- **Test helper** (`cl_reftable_write_block`) to reduce boilerplate in unit tests.
- **Fixes for vulnerabilities** discovered by fuzzing: out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`.

**Key fixes**:
- **Patch 3/12**: Fixes an infinite loop in `binsearch()` when the range is empty.
- **Patch 4/12**: Replaces `abort()` with `REFTABLE_FORMAT_ERROR` for invalid ref value types.
- **Patches 6–12/12**: Fix heap-buffer-overflows, out-of-bounds reads, and NULL dereferences in block parsing.

The series is **fully reviewed and ready for merging**, with all feedback addressed.

**Why it matters**: Reftable is a critical backend for large repositories, and this series proactively hardens it against corruption. The fuzzing infrastructure will help prevent regressions in this security-sensitive area.

---

### `git history` subcommands
Two new subcommands for `git history`—`drop` and `squash`—were merged this week, but both required **urgent post-merge fixes** for critical regressions.

#### `git history drop`
**Patrick Steinhardt**’s 11-patch series implementing `git history drop` (to remove a commit and replay its descendants onto its parent) was **merged after resolving a critical ref resolution dispute** in v8. The command includes:
- **Conflict detection**: Aborts if replaying descendants would result in conflicts or overwrite local changes.
- **Bare repository support**: Works in both bare and non-bare repositories.
- **Ref updates**: Moves branches pointing to the dropped commit to its parent, with configurable scope via `--update-refs=(branches|head)`.
- **Dry-run mode**: Supports `--dry-run` to preview ref updates.
- **561 lines of test coverage** in `t/t3454-history-drop.sh`.

**Critical dispute resolved**: Junio identified a **logical flaw in v7** where `find_head_tree_change()` searched for symbolic branch names in `result->updates[]` while `compute_pending_ref_updates()` filtered the array to contain only `HEAD` under `--update-refs=head`. The v8 fix adds `RESOLVE_REF_READING` to `refs_resolve_ref_unsafe()`, ensuring it returns `NULL` when `HEAD` cannot be resolved.

**Reset API modernization**: The series also modernizes the reset API, renaming `reset_head()` to `reset_working_tree()`, converting flags to an enum, adding dry-run mode, and making HEAD updates opt-in. The changes are **backward-compatible** and advance the `the_repository` removal effort.

**Why it matters**: `git history drop` is a powerful tool for history rewriting, and the series advances the `git history` command’s capabilities. The reset machinery refactoring is also a step toward modernizing Git’s internals.

#### `git history squash`
**Harald Nordgren**’s 4-patch series implementing `git history squash` (to fold a commit range into its oldest commit while preserving descendant history) reached **consensus on usability tweaks** and is ready for final review. The series includes:
- **Merge commit handling**: Rejects merges with external parents but allows fully contained merges.
- **Ref-handling**: Rejects operations with interior refs by default, with advice to use `--update-refs[=head]`.
- **`--reedit-message` template**: Adopts Phillip Wood’s finalized layout (summary list of subjects, grouped `fixup!`/`amend!` commits, minimal separators).
- **Recoverability concerns**: Matt Hunter and Phillip Wood highlighted that `git reset --hard` is insufficient for undoing `--update-refs` operations, as Git’s reflog lacks visibility into which refs were affected. Phillip suggested an **oplog (operations log)** to enable atomic undo of multi-ref operations.

**Why it matters**: `git history squash` fills a gap in Git’s history-editing toolkit, offering a more efficient alternative to `git rebase -i` for collapsing ranges. The recoverability discussion exposes a **systemic limitation** in Git’s recovery tooling.

---

### `git replay --linearize` post-merge issues
**Toon Claes**’s series adding `--linearize` to `git replay` was **merged to `master`**, but **three critical issues** were identified post-merge:
1. **Silent commit dropping regression** in single-branch replay with merge commits (Johannes Schindelin, **highest priority**).
2. **CLI design inconsistency** with `git rebase` (Patrick Steinhardt, **strategic concern**).
3. **Merge commit divergence handling** (Junio Hamano, Phillip Wood, **design limitation acknowledged**).

**Root cause of regression**: v5’s base-selection logic **inadvertently removed the `replayed_base` mechanism**, causing only the tip commit to be replayed while intermediate commits (including merges) are lost. Johannes provided a test case (`master~2..master`) showing v5 replays only the tip commit directly onto `--onto`, dropping the "Git 2.55-rc2" commit entirely.

**Design intent clarified**: Toon confirmed `--linearize` is intended to produce a **single linear sequence** regardless of input branches, even if this duplicates shared history. This aligns with Johannes’s preference for predictable behavior but diverges from `git rebase --no-rebase-merges`.

**Follow-up required**:
- **Urgent patch** to restore `replayed_base` logic or redesign multi-branch handling.
- **Documentation update** to clarify `--linearize` behavior.
- **Test cases** to expose merge commit divergence behavior.
- **CLI design decision**: Adopt `git rebase` syntax, diverge with clearer UX (e.g., `--replay-merges=<mode>`), or hybrid approach.

**Why it matters**: `git replay` is an experimental command, but these issues risk data loss and user confusion. The CLI inconsistency could limit future compatibility.

---

### Memory-leak fixes
**Jeff King** submitted a **9-patch series** plugging memory leaks in Git’s hash implementations when using non-default backends (OpenSSL SHA-256, libgcrypt). The leaks, discovered via `SANITIZE=leak`, affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. The series introduces a new `git_hash_discard()` primitive and systematically applies it to leaky code paths.

**Key discussion points**:
- **Naming**: Patrick Steinhardt suggests `git_hash_release()` for idiomatic consistency with Git’s existing resource-cleanup functions (e.g., `strbuf_release()`).
- **Idempotency**: Patrick and Brian M. Carlson argue for making `git_hash_discard()` idempotent to simplify error paths, citing historical pain with non-idempotent APIs (e.g., reference transactions). This would benefit Git’s Rust integration by simplifying the `Drop` implementation.

**Why it matters**: The debate touches on **API design principles** and **long-term maintainability**. The outcome could influence how Git designs future resource-cleanup functions, especially as Rust integration progresses.

---

### Coverity-driven leak fixes
**Johannes Schindelin** posted a 13-patch series addressing resource leaks and error-path bugs identified by Coverity in Git’s codebase. The series targets memory, file-descriptor, and process-handle leaks across core Git, including fixes for `load_one_loose_object_map` (loose.c), `start_command` (run-command.c), and the fsmonitor daemon’s startup sequence.

**Key resolutions**:
- **Patch 1/13**: Replaces an unsafe `errno`-based error check with `ferror(fp)` to reliably detect file-reading errors.
- **Patch 5/13**: Dropped entirely after Patrick Steinhardt identified it as ineffective.
- **Patch 7/13**: Clarifies ownership of allocations, resolving a completeness concern.
- **Patch 8/13**: Consolidates cleanup into a single exit path to eliminate a double-free risk.

The series is **fully reviewed and ready for merging**, with no open technical questions.

**Why it matters**: The series **systematically addresses technical debt** in Git’s error-handling paths, reducing the risk of resource leaks and crashes.

---

## In brief

> **`git cat-file --batch-command` remote object metadata support** -- Pablo Sabater’s GSoC project to add `remote-object-info` support to `git cat-file --batch-command` reached its 15th iteration and is **ready for merging**. The series allows clients to query object metadata (initially just size) from remote repositories without downloading full objects, using a new `remote-object-info` command that works with protocol v2 servers advertising the `object-info` capability.

> **`git refs` subcommands** -- Patrick Steinhardt’s 5-patch series adding `create`, `delete`, `update`, and `rename` to `git refs` was **merged after a final typo fix**. This consolidates reference manipulation under a unified interface.

> **Reftable quadratic-time behavior fix** -- Kristofer Karlsson’s patch exposes tombstones to iterator bounds checks, reducing runtime from **O(n²) to O(n)** (e.g., 8,000 refs: 14.93s → 0.145s). The fix was **merged after Patrick Steinhardt’s review**.

> **`excludes_file` migration into `repo_config_values`** -- Tian Yuchen’s series completed the **three-phase guardrail migration** (silent return → `BUG()` → no check) and was **merged into `next`**. The series eliminates the global `excludes_file` variable as part of the libification effort.

> **`git blame -b` output formatting fix** -- René Scharfe’s patch removes the extra hex digit reserved for an unused caret marker, aligning abbreviated commit hashes with `core.abbrev`. The fix was **confirmed by the original reporter (Laszlo Ersek)** and is ready for merging.

> **HTTP/HTTPS authentication regression in Git for Windows** -- A user reported that `http."<url>".allowNTLMAuth true` no longer works in Git for Windows 2.55.0.windows.1. Johannes Schindelin redirected the discussion to the Git for Windows issue tracker ([#6308](https://github.com/git-for-windows/git/issues/6308)).

> **`git gui` encoding mismatch fix** -- A patch fixes `git gui` failures on Windows when the home directory contains non-ASCII characters (e.g., EntraID/AzureAD profiles). The fix replaces `safe_exec` with `safe_open_command -encoding utf-8` to match `cygpath`’s UTF-8 output.

> **Meson build race fix** -- D. Ben Knoble’s patch restores `hook-list.h` to the `builtin_sources` list in `meson.build`, preventing a race condition that causes build failures when the header is missing.

> **CI PID limits for private GitHub repositories** -- Johannes Schindelin’s patch adjusts Dockerized CI jobs in private repositories to use explicit process/file limits (`--pids-limit=16384`, `--ulimit=nproc=16384`, `--ulimit=nofile=32768`), preventing resource exhaustion.

> **Bloom filter leak fixes** -- Jeff King’s 3-patch series plugs memory leaks in Bloom-filter code paths exposed by `GIT_TEST_COMMIT_GRAPH_CHANGED_PATHS=1`. The series was **endorsed by Junio Hamano** and is ready for merging.

> **`greplint.pl` test-suite linter** -- Michael Montalbo’s 6-patch series introducing `greplint.pl`, a linter that converts bare `grep` assertions to `test_grep` in the test suite, was **approved for merging**. The linter uncovered 10+ pre-existing test bugs during development.

> **`git rebase -i -x` design questions** -- Trevor Gross’s patch introducing a `-x` flag to `git rebase -i`’s `pick`, `reword`, and `edit` commands sparked a **substantive design discussion**. Junio Hamano raised fundamental questions about the feature’s alignment with rebase’s core semantics, arguing that rebase (which moves history) and cherry-pick (which duplicates history) serve fundamentally different workflows.

---

## Looking ahead

### Urgent follow-ups
- **`git replay --linearize` regression**: The merged v5 series has a **critical bug** where single-branch replay silently drops commits. Toon Claes’s v6 series is **ready for re-merge** to fix the issue.
- **`git history squash` recoverability**: The `--update-refs` behavior reveals a **systemic limitation** in Git’s recovery tooling. Phillip Wood’s oplog proposal is a **long-term architectural idea** to watch.

### Ongoing efforts
- **ODB abstraction**: Patrick Steinhardt’s work continues, with the `struct object_info` refactoring and `prepare()` callback generalization now in `next`. Future series will focus on **pluggable backends**.
- **Rustification**: Git 2.55.0 enables Rust by default, and the build system adjustments for macOS Universal Binaries are **ready for merging**. The timeline for Git 3.0 (where Rust may become mandatory) remains a topic to watch.
- **Reftable performance**: Jeff King identified **quadratic-time behavior** in the reftable backend during bulk ref deletion/re-creation. This could inform future optimizations.
- **CI improvements**: Patrick Steinhardt’s `GIT_TEST_LONG` CI improvements are **ready for merging**, expanding test coverage while reducing flakiness.

### Emerging topics
- **`git repo` GSoC project**: K Jayatheerth’s project to improve partial clone disk space recovery remains in the research/design phase, with updates communicated via external blog posts.
- **`git history` reorganization**: Patrick Steinhardt’s RFC v3 series to move `git history` into `lib/` remains stalled due to **skepticism about the discoverability rationale**. The debate may resurface as the command’s scope grows.
- **`git rebase` dropped-commit notes fix**: Phillip Wood’s 11-patch series is **queued in Junio’s tree** but carries a **known limitation**: the "edit" command still erroneously records dropped commits as rewritten. A follow-up patch is expected.