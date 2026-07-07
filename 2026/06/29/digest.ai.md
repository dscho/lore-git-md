The day in brief.
2026/06/29 was a **heavy, milestone-heavy Monday**—Git 2.55.0 shipped, Git for Windows 2.55.0 followed, and the “What’s cooking” report landed with 500+ commits and 100 contributors. **Do not miss**: the **reftable security-hardening series** (12 patches, fuzzing infra, all fixes merged) and **Patrick Steinhardt’s ODB abstraction work** (multiple series in flight, all conceptually approved). The day also saw **final resolutions** on long-running threads (`git history drop`, `git replay --linearize`, `paint_down_to_common()` optimizations) and **new regressions** (clock-skew in `git merge-base`, promisor remote hang) that are already fixed but need test coverage.

---

## Notable threads

### Reftable security hardening – 12 patches, fuzzing infra, all fixes merged
**What happened**: Patrick Steinhardt’s **12-patch series** systematically hardens Git’s reftable backend against maliciously corrupted files. The series adds **libFuzzer support to Meson**, a new fuzzer target (`oss-fuzz/fuzz-reftable.c`), and fixes for **OOB reads/writes, NULL derefs, uninitialized memory, and `abort()` calls** discovered by fuzzing. All patches are now **merged into `next`**, with Junio approving the majority as “obviously correct.” The fuzzing infrastructure is fully integrated into Git’s CI and oss-fuzz, ensuring no regressions.

**Why it matters**: Reftable files are not transferred over the network, but local corruption could still crash Git or leak memory. The hardening is **proactive and comprehensive**, addressing every vulnerability the fuzzer surfaced. The fuzzing infrastructure itself is a **long-term investment**—it can be reused for other subsystems and will help prevent future issues.

**Current state**: **Merged**. The series is complete, with no open technical concerns. The fuzzer has run for 2+ hours without surfacing new issues, and the fixes are paired with targeted unit tests. The only follow-up is **test coverage expansion**, but this is not blocking.

---

### ODB abstraction – multiple series, all conceptually approved
**What happened**: Patrick Steinhardt’s **ODB abstraction effort** saw **three series land or advance significantly**:
1. **`ps/odb-drop-whence` (7 patches)**: Replaces the `whence` field in `struct object_info` with an opt-in `struct odb_source_info`, enabling multi-source object resolution. The series is **merged into `next`**, with Junio’s conceptual approval (“Great”) and all review feedback addressed.
2. **`ps/odb-generalize-prepare` (3 patches)**: Generalizes the `reprepare()` callback into a `prepare()` callback with a flush flag, making it backend-agnostic. The series is **merged**, with no open concerns.
3. **`jt/receive-pack-use-odb-transactions` (6 patches)**: Refactors `git-receive-pack` to use ODB transactions instead of `tmp_objdir`, advancing backend agnosticism. The series is **queued for `next`**, with all feedback addressed in a planned v2.

**Why it matters**: The ODB abstraction is a **multi-year effort** to make Git’s object storage pluggable. These series are **foundational**—they eliminate global state, improve error handling, and prepare the codebase for alternative backends (e.g., reftable, cloud storage). The `receive-pack` refactoring is particularly significant, as it removes a major dependency on the “files” backend.

**Current state**:
- **`ps/odb-drop-whence`**: Merged.
- **`ps/odb-generalize-prepare`**: Merged.
- **`jt/receive-pack-use-odb-transactions`**: Queued for `next` pending v2 (addressing naming consistency and error-handling feedback).

---

### `git history drop` – 11 patches, merged after final review
**What happened**: Patrick Steinhardt’s **11-patch series** adds a `git history drop` subcommand to remove a commit and replay its descendants onto its parent. The series refactors the reset machinery to avoid touching HEAD or the index unnecessarily, adds a dry-run mode, and includes **561 lines of test coverage**. Junio’s final review surfaced a **logical flaw** in ref resolution (fixed in v7), and the series is now **merged**.

**Why it matters**: `git history drop` is a **new user-facing feature** that expands Git’s history-editing toolkit alongside `git rebase` and `git commit --amend`. The reset machinery refactoring is a **cleanup win**, eliminating global state and improving modularity. The dry-run mode is a **usability improvement**, allowing users to preview changes before committing.

**Current state**: **Merged**. The series is complete, with all feedback addressed. The only follow-up is **test helper consolidation**, but this is non-blocking.

---

### `git replay --linearize` – v6 posted, interface debate resolved
**What happened**: Toon Claes’s **6-patch series** adds a `--linearize` option to `git replay` to flatten merge commits, producing a linear history. The series reached **technical consensus** after addressing a regression (commit-dropping when replaying a single branch with merges) and documenting the behavioral difference from `git rebase --no-rebase-merges`. The **interface debate** (whether to mirror `git rebase`'s `--rebase-merges=<mode>` syntax) is now resolved in favor of `--linearize` as a standalone flag, with explicit justification in the commit message.

**Why it matters**: `--linearize` provides a **predictable, all-or-nothing flattening** alternative to `git rebase`'s more complex merge-replay modes. The series is **ready for final review**, with no open technical concerns. The interface choice (`--linearize` vs. `--rebase-merges`) was contentious but is now settled, avoiding future UX inconsistencies.

**Current state**: **Ready for final review**. Junio has not yet reported merge status, but the series is technically complete. The only open question is whether `--linearize` should also be added to `git history` (author sees no clear use case).

---

### `paint_down_to_common()` optimizations – v6 rebased, regression fix merged
**What happened**: Kristofer Karlsson’s **10-patch series** optimizes `paint_down_to_common()` to terminate early when one side of a merge-base query exhausts its commit queue. The series was **rebased on `kk/commit-reach-find-all-fix`** to resolve a procedural merge conflict and test failure, and the **regression fix** (commit-date fallback causing incorrect early exits) is now merged. The series is **ready for merging** as v6.

**Why it matters**: The optimization yields **100-1000x speedups** for asymmetric queries (e.g., repositories with import grafts or shallow histories). The regression fix ensures correctness when using v1 commit-graphs (no corrected commit dates), which are still common in older repositories. The series is **performance-critical** for large repositories and monorepos.

**Current state**: **Ready for merging**. The rebase is confirmed, and all feedback is addressed. The series is queued for `next` pending Junio’s final review.

---

### Clock-skew regression in `git merge-base` – 2-patch fix, merged for 2.55
**What happened**: A **regression in `git merge-base` (without `--all`)** was introduced by 93e5b1680e (April 2025), causing incorrect merge bases when all three conditions hold: (1) v1 commit-graph (no corrected commit dates), (2) clock skew in the topology, and (3) the correct merge base has a lower committer date than one of its ancestors. The **2-patch fix** gates the early-exit optimization on whether the queue is generation-ordered and includes a **regression test**. The series is **merged for 2.55**, with the in-flight `kk/merge-base-exhaustion` topic to be rebased on top later.

**Why it matters**: The bug affects **older repositories** (those that haven’t regenerated their commit-graph since 2021) with clock skew. The fix is **minimal and targeted**, addressing the regression without disrupting other optimizations. The test ensures no future regressions.

**Current state**: **Merged**. The series is complete, with no open concerns.

---

## In brief
- **Git 2.55.0 released**: 505 non-merge commits, 100 contributors, new features (parallel hooks, `git format-rev`, `git url-parse`, Linux fsmonitor), performance optimizations, and 50+ bug fixes.
- **Git for Windows 2.55.0 released**: Windows-specific fixes (infinite unlink loops, Git Bash shutdown, parallel checkout with FSCache), drops Windows 8.1 support.
- **`git history squash` – v7 posted**: Harald Nordgren’s 5-patch series adds a subcommand to fold a commit range into its oldest commit. All feedback addressed, including `--reedit-message` template alignment with `git rebase -i`.
- **`git refs` subcommands – merged**: Patrick Steinhardt’s 5-patch series adds `git refs create|delete|update|rename` subcommands, consolidating reference manipulation under a unified interface.
- **Promisor remote hang – fixed, needs test**: A zero-object push to a promisor remote caused `git receive-pack` to hang. The fix (commit d9982e8290) is merged, but **test coverage is missing**. Patrick Steinhardt offered to write a test if the reporter doesn’t.
- **`USE_NSEC` debate – runtime configuration proposed**: The discussion about `USE_NSEC` (nanosecond timestamp support) evolved into a proposal to **make it runtime-configurable** (e.g., `core.useNsec`), avoiding auto-detection challenges. No patch yet, but the direction is clear.
- **GSoC 2026 – partial clone pruning**: Siddharth Shrimali’s progress report outlines a `git maintenance run --task=gc-promisor` subcommand to safely prune unneeded promisor objects. Design is still open for feedback.

---

## On the radar
- **`git maintenance` promisor pruning**: Siddharth Shrimali’s GSoC work on `gc-promisor` is in the design phase. The proposed `.keep` file mechanism for re-fetching pruned objects needs review.
- **`USE_NSEC` runtime configuration**: Patrick Steinhardt’s proposal to make `USE_NSEC` runtime-configurable (e.g., `core.useNsec`) is gaining traction. A patch is expected soon.
- **Reftable performance optimizations**: Kristofer Karlsson’s **quadratic-time fix** for tombstone iteration is merged, but its real-world impact is still being evaluated. Patrick Steinhardt identified a **reproducible scenario** (bulk ref deletion without compaction) that could demonstrate the patch’s value.
- **`git replay --linearize` in `git history`**: Toon Claes left open whether `--linearize` should also be added to `git history`. No clear use case yet, but feedback is welcome.