# The Git Project Mailing List Daily Digest

**The day in brief.**
June 29, 2026 was a heavy-traffic day (106 emails across 29 threads) dominated by **feature finalization and regression fixes** ahead of the Git 2.55.0 release. The most consequential developments were the **resolution of a critical regression in `git replay --linearize`** (silent commit dropping when replaying single branches with merge commits) and the **finalization of Patrick Steinhardt’s `git history drop` series**, which adds a new subcommand for removing commits from history. Other notable threads included **security hardening for the reftable backend**, **ODB abstraction refactoring**, and **performance optimizations for merge-base computation**.

---

## Notable threads

### `git replay --linearize` regression: silent commit dropping
**Headline:** *Johannes Schindelin identified a critical regression in `git replay --linearize` (v5) that silently drops commits when replaying a single branch containing merge commits. The issue stems from the removal of `replayed_base` logic in `pick_regular_commit()`, which was intended to preserve linearized history. A follow-up patch is urgently needed before the 2.55.0 release.*

The thread began with Patrick Steinhardt questioning whether `--linearize` should mirror `git rebase`'s three-mode interface (`--no-rebase-merges`, `--rebase-merges`, and `--rebase-merges=rebase-cousins`). However, the discussion was overshadowed by Johannes Schindelin’s discovery of a **regression with silent data loss**: when replaying a range like `master~2..master`, v5 replays only the tip commit directly onto `--onto`, dropping intermediate commits (including merges). This differs from v4’s correct behavior and represents a **blocking issue for the 2.55.0 release**.

The root cause is the removal of `replayed_base` logic in `pick_regular_commit()`, which was intended to handle single-branch cases. Schindelin argued that `--linearize` must linearize *all* replayed commits into a single sequence to avoid user surprises when branches share history. The thread is now focused on a **follow-up patch** to restore the correct behavior, with Toon Claes expected to address it shortly. The regression highlights the complexity of merge commit handling in linearization and the need for robust testing of edge cases.

---

### `git history drop` series finalized
**Headline:** *Patrick Steinhardt’s 11-patch series adding `git history drop` (a subcommand to remove a commit and replay its descendants) is now complete and ready for merging. The series refactors the reset machinery, adds dry-run support, and resolves all prior review feedback, including code duplication and edge-case handling.*

The series introduces a new `git history drop` subcommand that removes a commit from history and replays its descendants onto its parent. Key features include:
- **Dry-run mode** (`--dry-run`) to preview changes without modifying the repository.
- **Refactored reset machinery** to avoid unnecessary HEAD/index updates, improving performance and correctness.
- **Strict input validation** and **conflict detection**, with clear error messages.
- **Support for bare repositories** and **preservation of local changes** during the operation.

Junio Hamano identified a **logical flaw** in the final patch: when `--update-refs=head` is used, `find_head_tree_change()` fails to detect HEAD movement because it looks for the branch name (e.g., `refs/heads/master`) in the `result->updates[]` array, which only contains `HEAD`. This could cause the command to skip necessary index/worktree updates. Steinhardt is expected to address this in a follow-up patch.

The series is a significant addition to the experimental `git history` command, offering a safer alternative to `git rebase -i` for removing commits. Its thorough test coverage (537 lines) and refactoring of the reset API make it a strong candidate for merging.

---

### Reftable security hardening
**Headline:** *Patrick Steinhardt posted a 12-patch series hardening Git’s reftable backend against maliciously corrupted files, including fixes for out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. The series also adds fuzzing infrastructure to prevent regressions.*

The series addresses vulnerabilities discovered via libFuzzer, including:
- **Heap-buffer-overflows** in log block reconstruction and block parsing.
- **Out-of-bounds reads** during key lookup and block initialization.
- **NULL pointer dereferences** when seeking to corrupted offsets.
- **Use of uninitialized memory** in binary search logic.

Each fix is paired with a unit test in `t/unit-tests/`, and the series introduces **Meson build support for libFuzzer**, enabling coverage-guided fuzzing in CI. The fuzzer now runs for 2+ hours without surfacing new issues, suggesting the fixes are comprehensive. The hardening is proactive, as reftable files are not stored remotely (exploitation requires local disk access), but the fixes are necessary for long-term robustness.

The series is part of the ongoing reftable effort led by Steinhardt and Karthik Nayak. Given the thorough test coverage and the critical nature of the fixes, it is likely to merge soon.

---

### ODB abstraction refactoring
**Headline:** *Patrick Steinhardt’s series refactoring `struct object_info` to replace the `whence` field with a more flexible `source` field (part of the ODB abstraction effort) received substantive review from Justin Tobler. The discussion centers on whether source tracking should be opt-in (current design) or centralized during `struct packed_git` initialization.*

The series replaces the coarse `whence` field in `struct object_info` with a new `struct object_info_source` that carries both source type and backend-specific data. This enables multi-source object resolution, a key requirement for pluggable ODB backends. Justin Tobler’s review raised architectural questions about the opt-in design, suggesting that the ODB source could instead be recorded once during `struct packed_git` initialization. Junio Hamano echoed this concern, questioning the necessity of the optional `source` parameter in `packed_object_info_with_index_pos()`.

The discussion remains open, but the series has Junio’s conceptual approval ("Great"). The opt-in design is intended to minimize overhead for callers that don’t need source information, but the reviewers’ feedback suggests a preference for simplicity over flexibility. Steinhardt may need to revisit the design in a future iteration.

---

### Merge-base performance optimization
**Headline:** *Kristofer Karlsson posted a two-patch bugfix series for a regression in `git merge-base` (without `--all`) introduced in April 2025. The fix gates an early-exit optimization on whether the queue is generation-ordered, preventing incorrect results in repositories with v1 commit graphs and clock skew.*

The regression occurs when all three conditions are met:
1. The repository uses a v1 commit graph (no corrected commit dates).
2. The commit topology has clock skew.
3. The correct merge base has a lower committer date than one of its ancestors.

The fix adds a `gen_ordered` flag to track whether the priority queue is generation-ordered, ensuring the early-exit optimization only fires when the queue is generation-ordered. The series includes a thorough test in `t6600-test-reach.sh` that constructs a clock-skew topology to demonstrate the bug.

Junio Hamano queried where the `gen_ordered` flag should live (in `paint_state` or as a local variable), but the series is otherwise ready for merging. The fix is small, well-scoped, and addresses a real-world edge case.

---

## In brief

- **`git history squash` series finalized:** Harald Nordgren’s series adding `git history squash` (to fold a range of commits into one) is now complete, with all review feedback addressed. Junio Hamano noted minor documentation and CLI style issues, but the series is otherwise ready for merging.
- **Shell completion dotfile handling:** Zakariyah Ali’s series to hide dotfiles by default in shell completion remains blocked on structural concerns. Junio Hamano nudged the author for a v4 iteration.
- **`trust_executable_bit` migration:** Tian Yuchen’s series migrating `trust_executable_bit` into `repo_config_values` (part of the `the_repository` removal effort) was merged.
- **Priority queue optimization:** Kristofer Karlsson’s series optimizing `paint_down_to_common()` for one-sided histories was approved by Junio Hamano, who coordinated the merging of a related API simplification patch.
- **Memoized commit traversal:** Tamir Duberstein’s series extending memoized commit traversal to all ref-filter commands remains stalled, with Junio Hamano checking for unresolved concerns.
- **`excludes_file` libification:** Tian Yuchen’s series migrating `excludes_file` into `repo_config_values` is ready for `next`, with Junio Hamano and Christian Couder agreeing on a three-phase guardrail migration.
- **`git history --reword` bugfix:** Junio Hamano’s patch fixing a file stream leak in `git history --reword` was updated to restore error checking on `fwrite()` after Patrick Steinhardt’s review.
- **OpenSSL CVE-2026-34182:** A security inquiry about CVE-2026-34182 in OpenSSL 3.5.6 (bundled with Git 2.54.0) concluded that Git is not exposed, as it does not use the affected CMS functionality. Git for Windows 2.55.0 will include the fixed OpenSSL 3.5.7.
- **Promisor remote hang:** Wei Hu reported a hang in `git receive-pack` when pushing a zero-object ref update into a promisor-configured repository. Patrick Steinhardt confirmed the issue was already fixed in Git 2.55.0.
- **Git 2.55.0 released:** Junio Hamano announced the release of Git 2.55.0, highlighting new features like parallel hooks, `git format-rev`, and performance optimizations. The release includes 505 non-merge commits from 100 contributors.

---

## On the radar

- **`git replay --linearize` follow-up:** Toon Claes is expected to post a follow-up patch to address the regression in `git replay --linearize` (silent commit dropping when replaying single branches with merge commits).
- **`git history drop` follow-up:** Patrick Steinhardt is expected to address Junio Hamano’s feedback about the logical flaw in `find_head_tree_change()`.
- **Reftable security hardening:** Patrick Steinhardt’s 12-patch series is ready for review and likely to merge soon.
- **ODB abstraction refactoring:** The discussion about the opt-in vs. centralized source tracking design in Patrick Steinhardt’s `ps/odb-drop-whence` series remains open.