## The day in brief

**2026-06-26 was a heavy but productive day on the Git mailing list**, with **86 emails across 17 threads** touching security hardening, performance optimizations, refactoring, and new features. The standout developments were **Patrick Steinhardt’s reftable security hardening series landing in "New Topics"**, **Taylor Blau’s ambitious RFC to combine `git repack --geometric` and `--cruft`**, and **Toon Claes’s v5 `git replay --linearize` series addressing all prior architectural concerns**. A **critical regression in Tian Yuchen’s merge-base optimization** was caught by the test suite, and **Junio Hamano’s "What’s cooking" report** set the stage for Git 2.55-rc2’s deep freeze.

---

## Notable threads

### Reftable security hardening lands in "New Topics"
**Topic: ps/reftable-hardening** -- Patrick Steinhardt’s **11-patch series** to harden Git’s reftable backend against maliciously corrupted files was officially listed in Junio’s "What’s cooking" report under **[New Topics]**. The series fixes out-of-bounds reads/writes, NULL pointer dereferences, and `abort()` calls during block parsing, and introduces **libFuzzer-based fuzzing infrastructure** to prevent regressions. Christian Couder provided **surface-level feedback** on test code hygiene, but the series is otherwise **ready for review**. This is a critical security effort for the reftable backend, which is increasingly central to Git’s storage layer.

### Taylor Blau’s RFC: Combining `--geometric` and `--cruft` repacks
**Topic: tb/repack-geometric-cruft** -- Taylor Blau sent a **10-patch RFC** to combine `git repack --geometric` and `--cruft`, which are currently mutually exclusive. The series introduces a new `--stdin-packs=follow-reachable` mode for `git pack-objects` to ensure only reachable objects from rolled-up packs are included, while unreachable objects are collected into cruft packs. The patches are **well-structured** (refactoring, plumbing, integration) and include **8 new tests**, but the timing (during the -rc phase) and Taylor’s upcoming role change suggest this may require **multiple iterations**. Junio Hamano already flagged a **potential correctness issue** in the reachability traversal logic, which could include unreachable objects in the output pack. This is a **major workflow improvement** for large repositories, but its complexity will likely draw close scrutiny.

### `git replay --linearize` v5 addresses all architectural concerns
**Topic: tc/replay-linearize** -- Toon Claes’s **v5 series** for `git replay --linearize` (a feature to flatten merge commits) **resolved all prior feedback**, including Junio Hamano’s and Patrick Steinhardt’s concerns about the boolean refactoring and interface scope. The series now includes **detailed code comments**, fixes a bug in `--onto` handling, and adds a new test case for replaying multiple divergent branches. Junio provided **surface-level feedback** on a `BUG()` message, but the series is **technically complete** and ready for final review. This is a key step toward implementing `git rebase` functionality on the server side.

### Critical regression in merge-base optimization caught by tests
**Topic: kk/merge-base-exhaustion** -- Tian Yuchen’s **v3 series** to optimize `paint_down_to_common()` for one-sided histories introduced a **critical regression** in patch 7/8, where an unconditional BUG assertion broke correctness when `min_generation` was not set. The regression was **caught by the test suite** (`t6600-test-reach.sh`), validating the series’ robust instrumentation. Kristofer Karlsson acknowledged the bug and plans to rework the logic for v4. The rest of the series (patches 1–6/8 and 8/8) remains **stable and ready for merging**, with Derrick Stolee praising the **trace2 instrumentation** and test infrastructure.

### `git history squash` v6 incoming to address edge cases
**Topic: hn/history-squash** -- Harald Nordgren’s **v5 series** for `git history squash` (a command to fold a commit range into its oldest commit) received **substantive feedback** from Phillip Wood and Junio Hamano about input validation and syntax flexibility. Harald plans a **v6** to reject single-commit ranges, clarify behavior for `fixup!`/`squash!` commits, and document merge commit handling. The core design (three-way merge, ref-handling logic) remains unchanged, but the **usability refinements** will be critical for merging. This is a **high-value feature** for users who want to collapse history without the repeated conflict stops of `git rebase`.

### Junio’s "What’s cooking" report: Git 2.55-rc2 in deep freeze
**Topic: What’s cooking in git.git (Jun 2026 #10)** -- Junio Hamano’s report listed **48 in-flight topics**, with only bugfixes and documentation updates eligible to graduate from `next` to `master` during the deep freeze. Key highlights:
- **New Topics**: `ps/reftable-hardening`, `ps/connected-generic-promisor-checks`, and `ps/odb-generalize-prepare` (all part of Patrick Steinhardt’s ODB abstraction effort).
- **Stalled**: `jt/config-lock-timeout`, `js/parseopt-subcommand-autocorrection`, and `cl/conditional-config-on-worktree-path` (all needing author responses).
- **Cooking**: `kk/merge-base-exhaustion` (expecting reroll), `hn/history-squash` (needs review), and `ps/refs-avoid-chdir-notify-reparent` (ready for `next`).
The report sets the stage for **Git 2.55’s final release** and clarifies which topics will land post-freeze.

---

## In brief

**`git cat-file --batch-command` remote object metadata** -- Pablo Sabater addressed Junio’s feedback on the `strtoul_szt()` helper, switching to `uintmax_t` for cross-platform safety. The series is **ready to merge**, with only a commit message update pending. This completes the **security-hardened implementation** of remote object metadata queries.

**`git history` message preparation fix** -- Junio Hamano sent a **bugfix patch** for the experimental `git history` command, fixing a file stream leak and inefficient file handling in `fill_commit_message()`. The patch is **tested and merge-ready**, addressing a Windows compatibility issue.

**Rustification: Loose object map validation** -- Feng Wu fixed a correctness issue in Git’s Rust codebase, adding explicit hash algorithm validation to `ObjectMap::insert()`. The patch is **well-motivated and tested**, advancing the Rustification effort.

**Libification: `excludes_file` migration** -- Tian Yuchen’s **v2 series** to move `excludes_file` into `struct repo_config_values` received **substantive feedback** from Junio and SZEDER Gábor. Junio questioned the temporary guard for uninitialized submodules, while Gábor caught a **build-breaking unused parameter**. A **v3 is expected** to address these issues.

**macOS CI hang: Apache timeout fix** -- Michael Montalbo confirmed that **increasing Apache’s `Timeout` directive** resolves the CI hangs in `t5551` and `t5559`. The discussion now centers on the optimal timeout value (5–10 minutes), with Junio and Peff endorsing the approach. This is a **pragmatic fix** for a long-standing CI flakiness issue.

---

## On the radar

**`ps/odb-drop-whence`** -- Patrick Steinhardt’s series to remove the `whence` field from `struct object_info` is **waiting for review**. This is a **foundational refactoring** for the ODB abstraction effort, enabling backend-agnostic object queries.

**`jt/receive-pack-use-odb-transactions`** -- Jonathan Tan’s series to refactor `git-receive-pack` to use ODB transactions is **waiting for author response**. This is a **critical step** toward making the receive-pack machinery backend-agnostic.

**`ty/migrate-excludes-file`** -- Tian Yuchen’s series to migrate `excludes_file` into `struct repo_config_values` is **blocked on feedback** about the temporary submodule guard. This is part of the **ongoing libification effort** to eliminate global state.