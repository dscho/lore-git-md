## The day in brief

June 30, 2026 was a **heavy but routine** day on the Git mailing list, with **94 emails across 23 threads**. The standout developments were **two major series landing in `next`**—Patrick Steinhardt’s **reftable hardening** and **ODB abstraction** work—while **usability discussions** (e.g., `git blame -b` output, `git history squash` recoverability) and **build system debates** (Rust/C interop, `USE_NSEC`) kept the list lively. No single thread dominated, but the **sequencer memory leak fix** and **repository setup refactoring** series also made significant progress.

---

## Notable threads

### Reftable hardening lands in `next`
**Subject**: [PATCH v3 0/12] reftable: security hardening against corrupted files
**Author**: Patrick Steinhardt

A **12-patch series** systematically hardening Git’s reftable backend against maliciously corrupted files has been **merged into `next`**. The patches fix out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage discovered via libFuzzer, and add fuzzing infrastructure to prevent regressions. Junio C Hamano approved the majority of the fixes as "obviously correct," and the series is now **ready for the next release cycle**.

The hardening is comprehensive but narrowly scoped: reftable files are not stored remotely, so exploitation requires local disk access. The fuzzer has run for 2+ hours without surfacing new issues, and the fixes replace crashes or undefined behavior with error returns. The series also includes a **new test helper** (`cl_reftable_write_block`) to reduce boilerplate in block corruption tests, and the fuzzing infrastructure is now integrated into Git’s CI.

**Why it matters**: This is a critical security and reliability milestone for the reftable backend, which is increasingly used in large-scale deployments. The fuzzing infrastructure may also be reused for other subsystems.

---

### ODB abstraction stack advances
**Subject**: [PATCH v1 0/2] odb: generalize reprepare() callback for pluggable ODBs
**Author**: Patrick Steinhardt

A **3-patch series** generalizing the `reprepare()` callback into a more flexible `prepare()` callback with an optional flush flag has been **approved and merged into `next`**. The refactoring enables `git grep` to work generically with pluggable ODB backends by decoupling cache invalidation from ODB preparation. Junio C Hamano explicitly approved the series, and Toon Claes’s **substantive review** (questioning downcasting and flag semantics) was fully resolved.

The series introduces `enum odb_prepare_flags` with a single member (`ODB_PREPARE_FLUSH_CACHES`) and replaces `reprepare()` with `odb_prepare()`. Backward compatibility is preserved via the `odb_reprepare()` wrapper. The changes touch 12 files, primarily in the `odb/` subsystem, and are part of Patrick Steinhardt’s ongoing ODB abstraction effort.

**Why it matters**: This is a foundational step toward making Git’s object storage layer truly pluggable, a long-term architectural goal. The series is self-contained and ready for release.

---

### Sequencer memory leak fix nears completion
**Subject**: [PATCH 0/11] sequencer: avoid copying notes from dropped commits during rebase
**Author**: Phillip Wood

An **11-patch series** fixing a long-standing bug where `git rebase` incorrectly copies notes from dropped commits to the current HEAD has **completed review and is queued for merging**. The series also addresses systemic issues in how the rebase machinery handles dropped commits, including **external merge strategy failures** and **command execution failures** (e.g., `git commit` failing to run).

The core fix (patch 11/11) introduces `PICK_RESULT_DROPPED` to `enum pick_result` and modifies `pick_one_commit()` to skip `record_in_rewritten()` for dropped commits. The series is **self-contained and tested**, with expanded coverage in `t3400-rebase.sh` and `t5407-post-rewrite-hook.sh`. Junio C Hamano has queued the series with minor typofixes, and Uwe Kleine-König’s **Tested-by** confirms the regression is resolved.

**Why it matters**: This is a **substantive, tested-level contribution** that refactors the sequencer’s control flow while fixing a real-world bug. The series also resolves three additional rebase bugs (final fixup cleanup, edit command handling, and skip command handling).

---

### Repository setup refactoring progresses
**Subject**: [PATCH 0/13] Separate repository discovery from setup
**Author**: Patrick Steinhardt

A **13-patch refactoring series** to separate repository *discovery* (finding the repository location and format) from *setup* (configuring the repository object) has **landed its second patch in `next`** after Junio C Hamano fixed a minor typo. The series introduces `struct repo_discovery` to hold discovery results, enabling distinct, self-contained phases and reducing scattered setup logic.

Key changes include:
- Moving the "bogus worktree" flag logic to `apply_repository_format()` (patch 2/13, now merged).
- Embedding `struct repository_format` into `struct repo_discovery` (patch 6/13).
- Extracting the discovery phase into `repo_discover()` (patch 10/13).
- Making the worktree path an explicit parameter to `init_db()` (patch 12/13).

The series is **foundational for the `the_repository` removal effort** and aligns with Patrick’s broader ODB abstraction goals. No behavior changes are intended, but the separation may expose edge cases in repository initialization.

**Why it matters**: This is a **major architectural refactor** that will simplify future work on repository configuration and modularization.

---

### `git blame -b` output formatting fix
**Subject**: git blame -b output formatting and commit hash abbreviation
**Author**: Laszlo Ersek (user report), René Scharfe (patch)

A **usability bug** in `git blame -b` (show boundary commits as blank) has been **fixed by René Scharfe**. The issue: `git blame -b` reserved an extra hex digit in commit hash abbreviations for a caret marker that was never displayed, causing non-boundary commit hashes to exceed the user’s `core.abbrev` setting by one character. This created friction in manual workflows (e.g., pasting hashes from `git blame -b` into `git rebase -i`).

The patch refactors the mark-handling logic in `blame.c` to count and print boundary/ignored/unblamable marks only when they are actually shown, ensuring the abbreviated commit hash length aligns with `core.abbrev`. The fix is **minimal, well-structured, and confirmed by the original reporter**, but lacks test coverage. Junio C Hamano proposed the targeted fix, and the patch is now **ready for integration**.

**Why it matters**: This is a **small but high-impact usability improvement** for developers working with monorepos or complex histories. The fix preserves the fixed-width nature of the output while eliminating a persistent annoyance.

---

### `git history squash` recoverability debate
**Subject**: [PATCH v7 0/5] replay: introduce --linearize option
**Author**: Harald Nordgren

A **usability discussion** about recoverability in the `git history` suite has surfaced a **systemic gap in Git’s design**: there is no way to atomically undo operations that move multiple branch refs (e.g., `git history squash --update-refs`). Phillip Wood and Matt Hunter noted that `git reset --hard` is insufficient if `--update-refs` retargets other branches, as Git’s reflog lacks visibility into which refs were modified.

Harald Nordgren proposed a **human-centered reflog** with visual hierarchy (e.g., indentation for sub-steps) to reflect the logical structure of actions, while Phillip Wood suggested **reflog transaction IDs** or a separate operations log to enable atomic undo. The discussion is **forward-looking** and not tied to the `squash` series, but it highlights a **recurring pain point** in Git’s UX.

**Why it matters**: This is a **latent architectural issue** that could inform future work on Git’s undo/redo system. The `git history` suite’s safety net is incomplete, and the discussion may inspire proposals for an oplog or similar mechanism.

---

## In brief

- **`git refs` subcommands merged**: Patrick Steinhardt’s **5-patch series** adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` has been **merged into `next`**. The series consolidates reference manipulation under a unified interface, improving discoverability without altering on-disk formats. A minor typo in `git refs create` was fixed in the final reroll.
  > **Files touched**: `builtin/refs.c`, `Documentation/git-refs.adoc`, and new test scripts (`t1464-refs-delete.sh` through `t1467-refs-rename.sh`).

- **Rust/C interop debate**: A **build system patch** to remove a spurious dependency between the Rust static library and the main Git C library was **rejected by Brian m. carlson**. The dependency is necessary for `cargo test` to work, as the Rust code in `src/hash.rs` links against C functions from `libgit.a`. The discussion highlights the **growing interdependence between Rust and C code** in Git.
  > **Files touched**: `Makefile`, `src/hash.rs`.

- **`USE_NSEC` runtime configuration proposed**: Patrick Steinhardt proposed converting `USE_NSEC` (sub-second file timestamp tracking) from a build-time option to a **runtime-configurable setting** (e.g., `core.useNsec`). Jeff King endorsed the idea but noted the **lack of reliable auto-detection** for filesystem timestamp granularity. The discussion remains in the design phase.
  > **Subsystems**: `git-config` machinery, `git-init`.

- **Test modernization**: Bryan B. Lima’s patch replacing raw test primitives (`test -f`, `test -d`) with descriptive helpers (`test_path_is_file`, `test_path_is_dir`) in `t/t7412-submodule-absorbgitdirs.sh` was **queued by Junio C Hamano**. The change improves test failure messages without altering behavior.
  > **Files touched**: `t/t7412-submodule-absorbgitdirs.sh`.

- **Memory leak fixes**: Jeff King’s **2-patch series** fixing leaks in `git format-patch` and the test harness was **merged in principle**. The patches are mechanical and low-risk, addressing small, self-contained leaks discovered opportunistically.
  > **Files touched**: `builtin/log.c`, `t/test-lib.sh`.

---

## On the radar

- **`git replay --linearize` interface design**: The debate over whether to use `--linearize` as a standalone flag or mirror `git rebase`’s `--rebase-merges=<mode>` syntax remains unresolved. Johannes Schindelin opposes the `git rebase` interface on usability grounds, while Patrick Steinhardt requires explicit justification for any divergence. The series is **technically complete** but blocked on this design question.
  > **Author**: Toon Claes.

- **Reftable quadratic-time behavior**: Jeff King identified a **quadratic-time scalability issue** in the reftable backend during bulk ref deletion and re-creation. The issue stems from tombstone iteration inefficiency and could be addressed by Kristofer Karlsson’s **merged fix**, but its real-world impact remains unresolved. Patrick Steinhardt suggested a **reproducible scenario** (bulk ref deletion without compaction) to validate the patch.
  > **Subsystem**: reftable backend (`reftable/merged.c`).

- **CI resource allocation**: The proposal to enable `GIT_TEST_LONG` for GitLab CI remains **on hold** pending further discussion about whether GitLab should duplicate GitHub’s coverage or focus on additional tests. Junio C Hamano suggested GitLab could run a **superset** of GitHub’s tests, but no concrete plan has been proposed. A **GitLab CI status badge** was added to `README.md` to improve visibility.
  > **Author**: Patrick Steinhardt.