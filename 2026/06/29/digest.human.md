The day in brief.
Monday, June 29, 2026 brought a **heavy but focused** day on the Git mailing list: 106 emails across 29 threads, with **one major release (Git 2.55.0) announced**, several long-running patch series reaching final review or merge, and a flurry of last-minute bugfixes and optimizations. The standout developments were the **Git 2.55.0 release**, the **finalization of the `git history drop` and `git history squash` subcommands**, and **security-hardening work for the reftable backend**. If you only have time for two things today, read about the **Git 2.55.0 release** and the **`git history` subcommands**—both mark significant milestones in Git’s evolution.

---

### Notable threads

#### Git v2.55.0 released
Junio C Hamano announced the official release of **Git v2.55.0**, capping a cycle that integrated **505 non-merge commits from 100 contributors**. Highlights include:
- **Parallel hook execution** (`hook.jobs` and per-event configuration).
- **New built-ins**: `git format-rev` (pretty-formatting revision expressions) and `git url-parse` (internal URL parsing).
- **Performance optimizations**: Faster revision traversal, reachability bitmaps, and sparse-index operations.
- **Rust support**: Now enabled by default (though still opt-out), with preparatory work in the xdiff/ codebase.
- **Linux fsmonitor daemon**: A native Linux implementation of the fsmonitor daemon, complementing existing Windows/macOS backends.
- **Breaking changes**: Stricter proxy URL validation and adjustments to sideband terminal control sequences (now disabled by default except for ANSI colors).

The release also includes over **50 bug fixes**, ranging from memory leaks to edge cases in `git bisect` and `git describe`. The changelog is organized into UI/workflow changes, performance improvements, and internal refactoring, reflecting a balanced focus on user-facing features and architectural modernization. **Weijie Yuan** noted a humorous inclusion in the contributor list ("Claude Sonnet 4.6"), but this was purely celebratory and did not affect the release.

---

#### `git history drop` and `git history squash` finalized
Two major new subcommands for the experimental `git history` built-in reached their final review stages today, marking a significant expansion of Git’s history-editing toolkit.

### `git history drop` (Patrick Steinhardt, 11 patches, v7)

This subcommand removes a commit from history and replays its descendants onto its parent, addressing a long-standing gap in Git’s history-editing capabilities. Key features:
- **Dry-run mode** (`--dry-run`) to preview changes without modifying the repository.
- **Ref-handling safety**: By default, the command rejects operations where refs point to commits inside the dropped range, with an advice message guiding users to `--update-refs=head` to explicitly retarget the current branch.
- **Reset API modernization**: The series refactors the reset machinery to avoid unnecessary HEAD/index updates, improving performance and correctness.
- **Conflict detection**: The command detects and aborts on conflicts, preserving local changes.

The series also includes a **critical bugfix** for a latent issue in the reset machinery (unconditional skipping of cache-tree updates, which could corrupt the index when resetting with a dirty index). **Junio C Hamano** identified a **logical flaw** in the ref-resolution logic (patch 10/11), which Patrick addressed in v7. The series is now **technically complete and ready for final review**, with no unresolved objections.

### `git history squash` (Harald Nordgren, 6 patches, v6)

This subcommand folds a commit range into its oldest commit while preserving descendant history, offering a more efficient alternative to `git rebase -i` for linearizing history. Key features:
- **`--reedit-message` flag**: Seeds the editor with a template identical to `git rebase -i`’s squash behavior, ensuring consistency for users.
- **Merge commit handling**: Rejects merges with external parents (to avoid ambiguity) but allows fully contained merges (all parents within the range).
- **Ref-handling safety**: Similar to `git history drop`, the command rejects operations with refs pointing to interior commits by default, advising users to use `--update-refs=head`.
- **Input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).

The series includes a **preparatory refactoring** of the reset API (adding a `message_template` parameter to `commit_tree_ext()`) and a **new test helper** (`cl_reftable_write_block`) to reduce boilerplate in test cases. **Phillip Wood** raised usability concerns about the `--reedit-message` template format and input validation, which Harald addressed in v6 by adopting `git rebase -i`’s squash-message template and adding stricter validation. The series is now **ready for final review**, with only minor documentation tweaks remaining.

Both subcommands advance the **`git history`** effort, which aims to provide a modern, built-in alternative to `git rebase` and `git filter-branch`. The `drop` and `squash` subcommands are particularly notable for their **safety mechanisms** (dry-run mode, ref-handling guards) and **performance optimizations** (avoiding unnecessary HEAD/index updates), which address common pain points in history editing.

---

#### Reftable backend security hardening (Patrick Steinhardt, 12 patches, v2)
Patrick Steinhardt posted a **v2 of his security-hardening series for the reftable backend**, addressing vulnerabilities discovered via fuzzing with libFuzzer. The series includes:
- **Fuzzing infrastructure**: Meson build support for libFuzzer and a new fuzzer target (`oss-fuzz/fuzz-reftable.c`).
- **Test helper**: `cl_reftable_write_block` to reduce boilerplate in unit tests for block corruption scenarios.
- **10 security fixes**: Addressing out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()` triggered by maliciously corrupted reftable files.

Key fixes include:
- **Heap-buffer-overflow in log block parser** (patch 6/12): Triggered when the inflated block size is smaller than the header size, leading to an out-of-bounds write during zlib decompression.
- **Out-of-bounds read in `reftable_block_init()`** (patch 7/12): Occurs when the block size exceeds the available data.
- **NULL pointer dereference in `reftable_block_type()`** (patch 11/12): Triggered when seeking to a bogus offset in a reftable table.
- **Heap-buffer-overflow in `reftable_table_new()`** (patch 12/12): Occurs when a truncated reftable file is too small to contain both the header and footer.

The series is **technically complete**, with all patches posted and reviewed. **Junio C Hamano** has approved several of the fixes as "obviously correct," and **Christian Couder** provided surface-level feedback on the test helper. The fuzzing infrastructure is now **fully integrated into Git’s CI**, ensuring these issues are unlikely to regress. This work is part of the broader **reftable backend effort**, which aims to provide a scalable, binary format for storing refs.

---

#### ODB abstraction and transaction refactoring (Patrick Steinhardt, Justin Tobler)
Two series targeting Git’s object database (ODB) abstraction and transaction system saw significant progress today:

### `ps/odb-drop-whence` (7 patches, v2)

This series removes the `whence` field from `struct object_info` and replaces it with an opt-in `struct object_info_source` carrying backend-specific data. The change is a **foundational step** toward multi-source object resolution and pluggable ODB backends. **Junio C Hamano** conceptually approved the series, and **Justin Tobler** provided substantive review, questioning the architectural rationale for the opt-in `source_infop` parameter. The series is **queued for merging**, with no unresolved objections.

### `jt/receive-pack-use-odb-transactions` (6 patches, v2 planning)

This series refactors `git-receive-pack` to use the ODB transaction API instead of `tmp_objdir`, advancing the ODB abstraction effort. **Patrick Steinhardt** provided substantive review, focusing on error-propagation correctness and behavioral changes. The series is **finalizing v2**, with all blocking feedback addressed, including:
- **Error-handling improvements**: Restoring explicit error checks for transaction setup.
- **Flag documentation**: Adding `ODB_TRANSACTION_RECEIVE` to `odb/transaction.h`.
- **Code organization**: Splitting the final patch and rewording user-facing error messages.

The series is **ready for v2 submission**, with no unresolved technical concerns.

---

### In brief
- **`git replay --linearize` (Toon Claes, 3 patches, v6)**: The series adding a `--linearize` option to `git replay` to flatten merge commits reached final review. **Patrick Steinhardt** raised a design question about whether the interface should mirror `git rebase`'s `--rebase-merges=<mode>` syntax, but the series remains on track for merging with the current `--linearize` flag.
- **`git refs` subcommands (Patrick Steinhardt, 5 patches, merged)**: The series adding `git refs create|delete|update|rename` subcommands was **merged into `next`**. The subcommands consolidate reference manipulation functionality previously scattered across `git-update-ref` and `git-symbolic-ref`, improving discoverability. **Junio C Hamano** approved the series, and **Toon Claes** provided surface-level feedback on usability (e.g., `--no-deref` behavior for symrefs).
- **`paint_down_to_common()` optimization (Tian Yuchen, 10 patches, v6)**: The series optimizing merge-base computation for one-sided histories was **technically complete and ready to merge** after a procedural rebase. **Kristofer Karlsson** and **Derrick Stolee** approved the removal of the commit-date fallback, which is now obsolete due to the side-exhaustion optimization.
- **`USE_NSEC` debate (D. Ben Knoble, Patrick Steinhardt, Jeff King)**: A long-running discussion about whether to flip the default of the `USE_NSEC` knob (for sub-second file timestamp tracking) or convert it to runtime-configurable reached a new phase. **Patrick Steinhardt** proposed always compiling nanosecond support into Git and exposing it via a runtime config knob (e.g., `core.useNsec`), while **Junio C Hamano** noted that `USE_NSEC` is optional (unlike `core.ignoreCase`), complicating the analogy. The discussion remains open, but the original Meson parity patch is **queued for merging**.
- **macOS CI hang fix (Jeff King, 2 patches, merged)**: The series addressing macOS CI hangs in `t5551` and `t5559` (caused by Apache’s `mod_http2` bug 70131) was **merged**. The fix increases Apache’s `Timeout` directive from 300 to 600 seconds and isolates the expensive "many-tags" test case into its own repository. **Patrick Steinhardt** proposed enabling `GIT_TEST_LONG` for GitLab CI, but the discussion remains open pending a policy decision about CI resource allocation.

---

### On the radar
- **`git maintenance` promisor object pruning (Siddharth Shrimali, GSoC)**: A Google Summer of Code project to enable safe pruning of unneeded promisor objects in partial clones is in the prototyping phase. The design proposes a `git maintenance run --task=gc-promisor` subcommand and a new config key (`maintenance.gcPromisor.enabled`), with a safety mechanism to mark pruned objects in the promisor pack’s `.keep` file for later re-fetch.
- **`greplint.pl` (Michael Montalbo, 6 patches, v4)**: The series introducing a test-suite linter to convert bare `grep` assertions to `test_grep` is **technically complete and approved for merging**. The linter has already uncovered **10+ pre-existing bugs** in the test suite, demonstrating its value. **Junio C Hamano** approved the series after addressing concerns about `# lint-ok:` exemptions.