# The Git Project Mailing List Daily Digest

**The day in brief**
June 26, 2026 (Friday) saw **86 emails across 17 threads**, marking a busy but focused day in Git development. The standout themes were **security hardening in the reftable backend**, **performance optimizations in merge-base calculations**, and **ongoing architectural refactoring** for ODB abstraction and repository modularity. Two long-running series—**`git replay --linearize`** and **`git history squash`**—reached key milestones, while a new RFC from Taylor Blau proposed combining `--geometric` and `--cruft` repack modes. Junio’s "What’s cooking" report set the stage for Git 2.55-rc2, with deep freeze in effect.

---

## Notable threads

### Reftable security hardening (ps/reftable-hardening)
**Patrick Steinhardt’s 11-patch series** to harden the reftable backend against corrupted files dominated the day, introducing **libFuzzer-based fuzzing infrastructure** and fixing **10 distinct vulnerabilities** (OOB reads/writes, NULL dereferences, uninitialized memory usage, and `abort()` calls). The series is **ready for review**, with Christian Couder providing surface-level feedback on test hygiene. The fuzzer, integrated into CI via `oss-fuzz`, is a first for Git and may be reused for other subsystems. This work is critical for the reftable backend’s stability, especially as it nears broader adoption.

### Merge-base performance optimization (kk/merge-base-exhaustion)
**Tian Yuchen and Kristofer Karlsson’s v3 series** to optimize `paint_down_to_common()` for one-sided histories saw **intense post-merge discussion** after a **critical regression** was discovered in patch 7/8. The regression—an unconditional BUG assertion for generation-number monotonicity—broke correctness when `min_generation` was unset, causing test failures in `t6600-test-reach.sh`. Junio ejected the series from `seen`, but the authors plan a **v4 rework** to restore the `min_generation` guard. The rest of the series (patches 1–6/8 and 8/8) remains stable, with **Derrick Stolee praising the trace2 instrumentation** and test infrastructure. The optimization yields **100-1000x speedups** for asymmetric histories, making it a high-impact change once the regression is resolved.

### ODB abstraction and repository modularity
Two threads advanced Patrick Steinhardt’s **ODB abstraction effort**:
1. **`ps/odb-generalize-prepare`** (2 patches): Generalizes the `reprepare()` callback into a `prepare()` callback with an optional `ODB_PREPARE_FLUSH_CACHES` flag, enabling `git grep` to work with pluggable ODB backends. **Toon Claes raised substantive questions** about downcasting and flag usage, but the series is otherwise ready for review.
2. **`ps/connected-generic-promisor-checks`** (5 patches): Refactors connectivity checks to search for promisor objects generically via the ODB interface, removing hardcoded packfile assumptions. **Christian Couder approved the final patch**, and the series is **ready for `next`**.

**Tian Yuchen’s series** to move `excludes_file` into `struct repo_config_values` (part of the libification effort) saw **mixed feedback**. Junio approved the memory management but questioned a **temporary guard** (`if (repo != the_repository)`) that bypasses BUG assertions, while **SZEDER Gábor identified a build-breaking unused parameter**. The series is **blocked on these issues** but is foundational for eliminating global state.

### `git replay --linearize` (tc/replay-linearize)
**Toon Claes’s v5 series** implementing `--linearize` for `git replay` **resolved all prior feedback**, including the controversial boolean refactoring and a bug in `--onto` handling. The series is **technically complete**, with Junio’s "Will merge to 'next'" approval contingent on the interface design question: whether to mirror `git rebase`'s three-mode design now or later. Toon outlined a future evolution (default: preserve topology, `--linearize`: flatten, `--no-merges`: abort, `--replay-cousins`: rebase cousins) but prefers incremental implementation. The series is **ready for merging** unless Junio raises new concerns.

### `git history squash` (hn/history-squash)
**Harald Nordgren’s v5 series** to fold commit ranges into a single commit saw **substantive review from Phillip Wood**, who identified **input validation gaps** (single-commit ranges, non-ancestor ranges) and **unclear behavior** around `fixup!`/`squash!` commits. Harald plans a **v6 with stricter validation and expanded tests**, but the core design (three-way merge, ref-handling logic) remains uncontested. The series is **blocked on these edge cases** but is otherwise ready for Junio’s final assessment.

### Combining `--geometric` and `--cruft` repacks (tb/midx-incremental-custom-base)
**Taylor Blau’s 10-patch RFC** proposes combining `git repack --geometric` and `--cruft` modes, which are currently mutually exclusive. The series introduces `--stdin-packs=follow-reachable` to `git pack-objects` and wires it into `git repack`, enabling geometric rollup of reachable packs while collecting unreachable objects into cruft packs. **Junio identified a correctness issue** in the two-phase traversal logic, which may retain unreachable tags and objects. The series is **complex and submitted late in the -rc cycle**, so it may require significant iteration before merging. However, it addresses a real workflow limitation and could streamline maintenance for large repositories.

---

## In brief

**`git cat-file --batch-command` remote object metadata** -- Pablo Sabater addressed Junio’s feedback on `strtoul_szt()` (patch 2/13), switching to `uintmax_t` to handle platforms where `unsigned long` is smaller than `size_t`. The series is **ready for merging** after a commit message update.

**`git replay --linearize`** -- Toon Claes’s v5 series resolved the boolean refactoring debate and fixed a `--onto` bug. The series is **technically complete** but awaits Junio’s final approval on interface design.

**Reference backend fixes (ps/refs-onbranch-fixes)** -- Patrick Steinhardt’s 11-patch series to resolve recursive initialization issues caused by `includeif.onbranch` is **merged into `next`** under the new topic name `ps/refs-onbranch-fixes`. The lazy-loading design was **approved by Jeff King (Peff)** and is poised to graduate to `master`.

**Git 2.55.0 translation coordination** -- Jiang Xin’s localization cycle is **closing tomorrow (June 27)**, with Junio reminding translators to finalize submissions.

**Gitk and git-gui quiet builds** -- Harald Nordgren’s series to align translation catalog generation with core Git’s quiet build conventions is **merged for gitk** and **superseded for git-gui** (equivalent changes already exist in Johannes Sixt’s fork).

**macOS CI hangs** -- Michael Montalbo identified the root cause as an **Apache `mod_http2` bug (70131)**, causing timeouts during `ls-refs` advertisements. The consensus is to **increase Apache’s `Timeout` value** as a pragmatic fix, with Peff endorsing a 600-second timeout.

**Rustification of loose object map** -- Feng Wu fixed a correctness issue in `ObjectMap::insert()` by validating hash algorithms, returning `ObjectMapInsertError` for mismatches. The patch is **ready for review**.

**`git history` reword fix** -- Junio fixed a file stream leak and inefficiency in `fill_commit_message()`, addressing a Windows compatibility issue. The patch is **tested and merge-ready**.

**`git branch --delete-merged`** -- **Hannes Nordgren’s series** to delete local branches already merged to their remote-tracking branch is **cooking in `next`** and needs review.

---

## On the radar

- **`ps/odb-drop-whence`** (7 patches): Removes the `whence` field from `struct object_info`, making backend-specific information opt-in. **Needs review**.
- **`jt/receive-pack-use-odb-transactions`** (6 patches): Refactors `git-receive-pack` to use ODB transactions instead of `tmp_objdir`. **Waiting for author response**.
- **`ad/gpg-strip-cr-before-lf`** (1 commit): Fixes GPG/SSH signature parsing for mixed CRLF/LF signatures. **Needs review**.
- **`hn/branch-push-slip-advice`** (3 patches): Adds advice for misplaced slashes in `git push` and `git branch`. **Waiting for author response**.
- **`ty/migrate-trust-executable-bit`** (3 patches): Moves `core.trustctime` and `core.filemode` into `struct repo_config_values`. **Needs review**.
- **`mm/line-log-limited-ops`** (7 patches): Limits `--stat`, `--check`, and `-G` to the specified range in `git log -L`. **Expecting reroll**.