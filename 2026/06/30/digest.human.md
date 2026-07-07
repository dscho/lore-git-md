Here is the digest for **June 30, 2026** (UTC), covering 94 emails across 23 active threads.

---

### The day in brief
A busy but focused day on the Git mailing list, with **performance optimizations, memory leak fixes, and foundational refactoring** dominating the traffic. The most consequential development: **Phillip Wood’s 11-patch series fixing a long-standing sequencer bug** (notes incorrectly copied from dropped commits during rebase) is now **technically complete and queued for merging**, resolving a systemic issue that had lingered since 2016. Other highlights include **Patrick Steinhardt’s ODB abstraction work** advancing with multiple series merged or approved, **Jeff King’s targeted leak fixes** in `git format-patch` and the test harness, and **a usability fix for `git blame -b`** that eliminates a persistent annoyance in manual workflows. The day also saw **CI infrastructure improvements**, **reftable hardening**, and **ongoing discussions about Rust integration** and repository setup refactoring.

---

### Notable threads

#### **Sequencer bugfix: notes from dropped commits no longer copied during rebase**
**Author**: Phillip Wood
**Series**: 11 patches, now **queued for merging**
**Problem**: When `git rebase` drops a commit (e.g., due to empty changes or fixup sequences), the sequencer incorrectly copies that commit’s notes to the current HEAD, corrupting the repository’s note history. The issue also affected post-rewrite hooks, which received misleading information about rewritten commits.
**Solution**: Phillip’s series introduces a new `PICK_RESULT_DROPPED` enum member and modifies `pick_one_commit()` to skip `record_in_rewritten()` for dropped commits. The series also fixes three additional rebase bugs: final fixup cleanup, edit command handling, and skip command handling. Test coverage is expanded to catch regressions in both the "apply" and "merge" backends.
**Status**: **Technically complete** and queued by Junio C Hamano. The series is self-contained, addresses all prior feedback, and carries Uwe Kleine-König’s `Tested-by`. A known limitation (the "edit" command still erroneously records dropped commits as rewritten) is documented and flagged for a separate fix.
**Key files**: `sequencer.c`, `t3400-rebase.sh`, `t3404-rebase-interactive.sh`, `t5407-post-rewrite-hook.sh`.
**Why it matters**: This is a **systemic fix** for a long-standing bug that has caused silent corruption in note histories. The series also improves the robustness of the rebase machinery, particularly when using external merge strategies or handling command failures.

---

#### **ODB abstraction: `ps/odb-generalize-prepare` approved for `next`**
**Author**: Patrick Steinhardt
**Series**: 3 patches, **approved and ready for `next`**
**Problem**: The `reprepare()` callback in the ODB subsystem was too rigid, conflating cache invalidation with ODB preparation. This blocked `git grep` from working with pluggable ODB backends.
**Solution**: The series generalizes `reprepare()` into a `prepare()` callback with an optional `ODB_PREPARE_FLUSH_CACHES` flag, enabling backend-agnostic preparation. The new `odb_prepare()` function pre-opens ODB sources, and the `packed` backend is updated to use the flag correctly.
**Status**: **Approved by Junio C Hamano** after Toon Claes’s substantive review was satisfied. The series is now ready to advance to `next`.
**Key files**: `odb/`, `builtin/grep.c`, `midx.c`, `packfile.c`, `odb.h`, `odb.c`.
**Why it matters**: This is a **foundational step** in Patrick Steinhardt’s ODB abstraction effort, enabling `git grep` to work with pluggable backends. The refactoring is minimal and well-tested, with no behavior changes.

---

#### **Memory leak fixes: `git format-patch` and test harness**
**Author**: Jeff King (Peff)
**Series**: 2 patches, **merged in principle**
**Problem**: A memory leak in `git format-patch` (since 2016) and a test harness issue where LeakSanitizer (LSan) output was misdirected to stdout, causing TAP parse errors with `prove`.
**Solution**:
- Patch 1/2: Redirect LSan output to stderr (descriptor 4) in `t/test-lib.sh`, fixing TAP parse errors.
- Patch 2/2: Add a `release_revisions(&revs)` call in `prepare_bases()` (`builtin/log.c`) to free a `rev_info` struct allocated for a custom revision walk.
**Status**: **Ready for merging**. Patrick Steinhardt confirmed the `format-patch` fix is correct, and the test harness change aligns with existing patterns. Peff also proposed **CI improvements** to enable LSan for the `linux-TEST-vars` job, benchmarking the overhead at ~3x slowdown.
**Key files**: `builtin/log.c`, `t/test-lib.sh`.
**Why it matters**: These are **targeted, low-risk fixes** that improve memory hygiene and test infrastructure. The `format-patch` leak was discovered opportunistically and demonstrates the value of ongoing leak detection efforts.

---

#### **`git blame -b` usability fix: no more extra hex digit**
**Author**: René Scharfe
**Patch**: Standalone, **ready for integration**
**Problem**: `git blame -b` (show boundary commits as blank) reserved an extra hex digit in commit hash abbreviations for a caret marker that was never displayed, causing non-boundary commit hashes to exceed the user’s `core.abbrev` setting by one character. This created friction in manual workflows (e.g., pasting hashes from `git blame -b` into `git rebase -i`).
**Solution**: The patch refactors the mark-handling logic in `blame.c` to count and print boundary/ignored/unblamable marks only when they are actually shown. This ensures the abbreviated commit hash length aligns with `core.abbrev`. The fix introduces helper functions (`print_marks()`, `count_marks()`) and dynamically calculates mark columns.
**Status**: **Confirmed by the original reporter (Laszlo Ersek)** with a `Tested-by` trailer. The patch is minimal, well-structured, and addresses the core problem directly. **Test coverage is still needed** to verify edge cases (e.g., multiple marks per line).
**Key files**: `blame.c`, `Documentation/git-blame.adoc`, `t/t8002-blame.sh`.
**Why it matters**: This is a **small but impactful usability fix** that eliminates a persistent annoyance in manual workflows. The patch preserves the fixed-width nature of the output while aligning with user expectations.

---

#### **Repository setup refactoring: separating discovery from setup**
**Author**: Patrick Steinhardt
**Series**: 13 patches, **initial submission complete**
**Problem**: Git’s repository setup code intertwines *discovery* (finding the repository location and format) with *setup* (configuring the repository object), making it harder to consolidate repository configuration logic into `repo_init()`.
**Solution**: The series introduces a new `struct repo_discovery` to hold discovery results, enabling distinct, self-contained phases. Key changes include:
- Extracting the discovery phase into `repo_discover()` (patch 10/13).
- Embedding `struct repository_format` into `struct repo_discovery` (patch 6/13).
- Moving the repository prefix from `startup_info` to `struct repository` (patch 7/13).
- Making the worktree path an explicit parameter to `init_db()` (patch 12/13).
- Removing `set_git_work_tree()` from the public API (patch 13/13).
**Status**: **Initial submission complete**. Junio C Hamano queued patch 2/13 after fixing a typo in its commit message. No substantive concerns have been raised, but the series is still early in review.
**Key files**: `setup.c`, `repository.h`, `setup.h`, and built-ins (`clone`, `init-db`, `rev-parse`).
**Why it matters**: This is a **foundational refactor** that paves the way for further `the_repository` removal and ODB abstraction work. The separation of discovery and setup phases will make the codebase more modular and easier to reason about.

---

### In brief

#### **Prio-queue optimization merged**
**Author**: Kristofer Karlsson
**Series**: 4 patches, **merged**
**Problem**: Three independent `lazy_queue` implementations (in `describe.c`, `commit.c`, `show-branch.c`) were replaced with a unified core optimization, reducing code duplication while delivering 1.7–2.7% speedups on traversal-heavy operations.
**Key files**: `prio-queue.c`, `commit.c`, `describe.c`, `show-branch.c`.
**Follow-up**: The author will reassess `kk/prio-queue-cascade-sift` for continued relevance.

---

#### **`git replay --linearize` interface debate**
**Authors**: Toon Claes, Johannes Schindelin, Patrick Steinhardt
**Problem**: The `--linearize` option for `git replay` (which flattens merge commits) uses a standalone flag, diverging from `git rebase`’s `--rebase-merges=<mode>` syntax. The debate centers on whether to mirror `git rebase`’s interface or retain the current design.
**Status**: **Unresolved**. Johannes Schindelin opposes mirroring `git rebase`’s syntax, calling it user-hostile, while Patrick Steinhardt requires explicit justification for any divergence. The regression in `--linearize` behavior (commit-dropping when replaying a single branch with merges) has been fixed, but the interface design remains contentious.

---

#### **Memory leaks in `git history reword` and slab management**
**Authors**: Kaartic Sivaraam, Jeff King
**Problem**: Memory leaks in `git history reword` and broader slab management gaps, particularly when commit-graph bypasses raw object parsing.
**Status**: **Fixed for `git history reword`**; the `t4014` leak (topological walk slab) was fixed in a separate patch. Peff identified the commit-graph as the primary slab-cache bypass mechanism and noted that reflog writes (unbuffered) consume ~70ms (54% of runtime) for 50,000 refs. **Follow-up**: Peff discovered a **quadratic-time behavior** in the reftable backend during bulk ref deletion/re-creation, which is now the focus of further investigation.

---

#### **`git history squash` feature complete**
**Author**: Harald Nordgren
**Series**: 5 patches, **v7 finalized**
**Problem**: Efficiently collapse a range of commits into one while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach.
**Solution**: The `git history squash` subcommand folds a range of commits into the oldest commit and replays descendants. The series includes `--reedit-message` (aligned with `git rebase -i`’s squash behavior) and `--update-refs=head` to retarget the current branch.
**Status**: **Technically complete** and ready for Junio’s final review. All prior feedback addressed, including interface design, merge commit handling, and input validation.
**Key files**: `builtin/history.c`, `sequencer.c`, `sequencer.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
**Open questions**: Whether `--reedit-message` should be the default (Phillip Wood advocates for this as a commit hygiene measure).

---

#### **Refs subcommands merged**
**Author**: Patrick Steinhardt
**Series**: 5 patches, **merged**
**Problem**: Reference manipulation functionality was scattered across `git-update-ref`, `git-symbolic-ref`, and other commands.
**Solution**: The series adds four new subcommands to `git refs`: `delete`, `update`, `create`, and `rename`. The `create` subcommand provides explicit atomic creation with "must not exist" semantics, while `update` mirrors `git update-ref`’s implicit creation behavior.
**Status**: **Merged into `next`**. Junio C Hamano signed off on the final v3 reroll, which fixed a typo in the `git refs create` error message.
**Key files**: `builtin/refs.c`, `Documentation/git-refs.adoc`, test scripts (`t1464-refs-delete.sh` through `t1467-refs-rename.sh`).

---

#### **Reftable hardening merged**
**Author**: Patrick Steinhardt
**Series**: 12 patches, **merged**
**Problem**: Fuzzing with libFuzzer uncovered security vulnerabilities in the reftable backend, including out-of-bounds reads/writes, NULL pointer dereferences, and calls to `abort()`.
**Solution**: The series fixes all discovered issues and adds a libFuzzer-based fuzzer to prevent regressions. The fuzzing infrastructure is now integrated into Git’s CI.
**Status**: **Merged**. Junio C Hamano approved the test helper (patch 5/12), and the series is now fully integrated.
**Key files**: `reftable/`, `oss-fuzz/fuzz-reftable.c`, `meson.build`.

---

#### **CI improvements: GitLab badge and LSan for `linux-TEST-vars`**
**Authors**: Patrick Steinhardt, Jeff King
**Problem**: GitLab CI results were not easily discoverable, and LeakSanitizer (LSan) was not enabled for the `linux-TEST-vars` job.
**Solution**:
- **GitLab badge**: A one-line addition to `README.md` linking to the GitLab CI pipelines page. **Merged**.
- **LSan for `linux-TEST-vars`**: Peff proposed enabling LSan for this job, benchmarking the overhead at ~3x slowdown. The proposal is **agreed upon** but not yet implemented.
**Status**: GitLab badge **merged**; LSan proposal **pending implementation**.

---

#### **Rust integration: Makefile dependency debate**
**Authors**: Jan Palus, Brian m. carlson
**Problem**: A Makefile dependency forces the Rust static library to rebuild whenever the main Git C library changes, but the Rust code does not link against the C library at build time.
**Solution**: Jan Palus proposed removing the dependency, but Brian m. carlson **opposed the patch**, explaining that the dependency is necessary for `cargo test` (the Rust code in `src/hash.rs` links against C functions from `libgit.a`). The absence of this dependency in the Meson build system is now considered an oversight.
**Status**: **Patch rejected**. The discussion highlights the growing interdependence between Rust and C code in the project.

---

### On the radar
- **`USE_NSEC` runtime configuration**: Patrick Steinhardt proposed converting the build-time `USE_NSEC` option to a runtime config knob (e.g., `core.useNsec`), but Peff noted the lack of reliable auto-detection. The discussion is **ongoing**.
- **Reftable performance**: Peff discovered a **quadratic-time behavior** in the reftable backend during bulk ref deletion/re-creation. The issue is reproducible with `git for-each-ref | git update-ref --stdin` and stems from tombstone iteration inefficiency. **Follow-up**: Kristofer Karlsson’s patch exposes tombstones to iterator bounds checks, but its real-world impact remains unresolved.
- **`git clone` sparse-checkout paths**: Pushkar Singh’s RFC for `--only`/`--except` options to specify sparse-checkout paths during clone received **cautious feedback** from Peff, who suggested a file-based patterns approach instead. The proposal is **stalled** pending engagement from sparse-checkout domain experts (e.g., Derrick Stolee).