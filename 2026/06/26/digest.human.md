## The day in brief

June 26, 2026 was a **heavy but productive** day on the Git mailing list, with **86 emails across 17 threads**. The standout developments were **two major series landing in `next`**—Patrick Steinhardt’s ref backend fixes (`ps/refs-onbranch-fixes`) and the `paint_down_to_common()` optimization (`kk/merge-base-exhaustion`)—while **Taylor Blau’s RFC to combine `--geometric` and `--cruft` repacks** sparked lively discussion. A **regression in the merge-base series** and **ongoing debate over `git history squash`** kept reviewers busy, but the overall tone was collaborative, with most threads advancing toward resolution.

---

### Notable threads

#### **`git cat-file --batch-command` remote object metadata series lands**
**Thread:** [GSoC v14] `git cat-file --batch-command` remote object metadata
**Author:** Pablo Sabater
**Status:** **Ready for `master`** after addressing final nits

Pablo Sabater’s **14-iteration security-hardened series** to implement `git cat-file --batch-command` for querying remote object metadata is now **complete and uncontroversial**. Today’s traffic focused on **minor documentation and type-safety fixes** for the `strtoul_szt()` helper (Junio Hamano, Karthik Nayak, Chandra Pratap), which converts strings to `size_t` with full error handling. The series introduces **dynamic format placeholder validation** (e.g., `%(objectsize)`) that adapts to server capabilities, a **memory leak fix** in `disconnect_helper()`, and **protocol v2 support** for the new `remote-object-info` command. All substantive feedback has been addressed, and the series is **poised to graduate to `master`** in the next integration cycle. The only remaining action is a **commit message tweak** for `strtoul_szt()` to clarify its return type.

---

#### **`git replay --linearize` v5 resolves architectural concerns**
**Thread:** [PATCH v5 0/3] `replay: introduce --linearize` option
**Author:** Toon Claes
**Status:** **Ready for `next`** after addressing Junio’s behavioral note

Toon Claes’s **v5 series** to add `--linearize` to `git replay` (flattening merge commits to match `git rebase --no-rebase-merges`) **resolved all prior architectural concerns** by reverting the controversial `enum replay_mode` refactoring and adding **detailed code comments** to clarify base-commit selection logic. The series now includes a **bug fix** for `--onto` handling with divergent branches and **expanded test coverage** for edge cases. Junio Hamano raised a **behavioral difference** with `git rebase`: where rebase rewrites both branches of a divergent merge, `replay --linearize` drops one branch entirely. Toon acknowledged the discrepancy and proposed documenting it, while Junio suggested adding a test to lock in the current semantics. The series is **technically complete** and likely to land in `next` shortly, with the behavioral note addressed in a follow-up.

---

#### **Ref backend lazy-loading series merged to `next`**
**Thread:** [PATCH v6 00/11] refs: avoid chdir-notify reparenting
**Author:** Patrick Steinhardt
**Status:** **Merged to `next` as `ps/refs-onbranch-fixes`**

Patrick Steinhardt’s **11-patch refactoring series** to resolve recursive initialization issues in Git’s reference backend (caused by `includeif.onbranch` conditions) **landed in `next`** after **Junio Hamano’s explicit merge signal**. The series replaces an earlier workaround with a **lazy-loading design** that defers write-config parsing until the first write operation, eliminating early config reads that could trigger recursion. Key changes include:
- **Deferred write-config parsing** (e.g., `core.logAllRefUpdates`, `reftable.blockSize`) via new backend-specific APIs.
- A **recursion guard** in `get_main_ref_store()` to prevent re-entrant initialization.
- **Fixes for latent memory leaks** in `chdir_notify` and `repo_clear()`.
The series is **foundational for ODB abstraction** and reftable integration, with **no user-visible behavior changes** unless `includeif.onbranch` is used. Jeff King (Peff) and Justin Tobler provided **final review approvals**, and the topic is now **poised to graduate to `master`**.

---

#### **`paint_down_to_common()` optimization hits regression, v4 planned**
**Thread:** [PATCH v3 0/8] `commit-reach: optimize paint_down_to_common()` for one-sided histories
**Author:** Tian Yuchen
**Status:** **Regression identified; v4 expected**

Tian Yuchen’s **v3 series** to optimize merge-base calculations by terminating early when one side’s commit queue is exhausted **hit a critical regression** in patch 7/8. The issue: an **overzealous BUG assertion** widened to fire unconditionally (not just when `min_generation` is set) broke correctness in some commit-graph modes. Junio Hamano **ejected the series from `seen`** after the test suite caught the failure in `t6600-test-reach.sh`. Kristofer Karlsson (co-author) acknowledged the bug and plans a **v4 rework** to either revert the problematic change or unify halt conditions while preserving correctness. The rest of the series—**edge-case tests, trace2 instrumentation, and documentation**—remains **stable and ready for merging** once the regression is fixed. Derrick Stolee praised the **test infrastructure enhancements** (e.g., deterministic step-count assertions) as "clean" and a strong regression guard.

---

#### **`git history squash` v6 expected after edge-case feedback**
**Thread:** [PATCH v5 0/4] `history: introduce squash command`
**Author:** Harald Nordgren
**Status:** **v6 expected to address input validation gaps**

Harald Nordgren’s **v5 series** to add `git history squash` (collapsing a commit range into its oldest commit) **received substantive feedback** from Phillip Wood on **input validation and usability**. Key concerns:
- The command accepts **single-commit ranges** (e.g., `@^!`) where squashing is meaningless.
- It allows **non-ancestor ranges** (e.g., `origin/seen^2^!` from `master`) without warning.
- Behavior with `fixup!`/`squash!`/`amend!` commits and **merge commits with external parents** is undocumented.
Harald agreed to tighten validation and clarify edge cases in **v6**, while Junio Hamano and Phillip Wood debated whether the command should support **two separate arguments** (e.g., `git history squash ^:/base :/tip`) for flexibility. The core **three-way merge approach** remains uncontested, but the interface may evolve to better match `git rebase`’s flexibility.

---

#### **macOS CI hangs: Apache timeout adjustment emerges as fix**
**Thread:** macOS CI hang in t5551/t5559 – root cause and fix
**Author:** Michael Montalbo
**Status:** **Consensus on timeout increase; patch forthcoming**

The **long-running CI hang investigation** reached a turning point: **Apache’s 300-second timeout** is confirmed as the root cause of `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. Michael Montalbo and Patrick Steinhardt reproduced the issue locally, and Jeff King (Peff) endorsed **increasing Apache’s `Timeout` to 600 seconds** as a pragmatic fix. The proposal avoids client-side mitigations (e.g., `http.lowSpeedLimit`) and aligns with the project’s preference for addressing flakes at the source. A **patch to adjust the timeout** in `t/lib-httpd.sh` is expected shortly, with no further changes to the test or CI configuration.

---

#### **RFC: Combining `--geometric` and `--cruft` repacks**
**Thread:** [RFC PATCH 00/10] `repack: combine --geometric and --cruft`
**Author:** Taylor Blau
**Status:** **Active discussion; complexity may delay merging**

Taylor Blau’s **10-patch RFC** to combine `git repack --geometric` and `--cruft` modes **sparked immediate discussion**, with Junio Hamano flagging a **potential correctness issue** in the new `--stdin-packs=follow-reachable` mode. The series introduces:
- **`--stdin-packs=follow-reachable`**: A two-phase traversal that includes only objects reachable from refs *and* present in included packs.
- **Refs snapshot synchronization**: Ensures consistency between `pack-objects` and the MIDX bitmap writer.
- **Geometric split interpretation**: The cruft writer now respects the split point, excluding packs above it from cruft collection.
Junio’s review highlighted that the current implementation may **over-retain unreachable tags**, risking inclusion of unreachable objects in the output pack. The series is **well-motivated** (streamlining maintenance workflows) but **complex**, and its timing (submitted during the -rc phase) suggests it may require **additional iterations** before merging. Taylor noted his upcoming role change but plans to continue contributing.

---

### In brief

- **`ps/reftable-hardening` (Patrick Steinhardt):** 11-patch series hardening the reftable backend against corrupted files (OOB reads/writes, NULL dereferences) **needs review**. Includes **libFuzzer-based fuzzing infrastructure** for regression prevention.
- **`ps/odb-generalize-prepare` (Patrick Steinhardt):** Generalizes the `reprepare()` callback into a `prepare()` callback with an `ODB_PREPARE_FLUSH_CACHES` flag, enabling `git grep` to work with pluggable ODB backends. **Needs review**.
- **`ps/connected-generic-promisor-checks` (Patrick Steinhardt):** Refactors connectivity checks to search for promisor objects generically via the ODB interface. **Ready for `next`**.
- **`ty/migrate-excludes-file` (Tian Yuchen):** Moves `excludes_file` into `struct repo_config_values` as part of the libification effort. **Junio raised concerns about a temporary guard** (`if (repo != the_repository)`) masking bugs; v3 expected.
- **Rustification (Feng Wu):** Fixes a correctness issue in `ObjectMap::insert()` by validating hash algorithms. **Uncontroversial**.
- **`jc/history-message-prep-fix` (Junio Hamano):** Fixes a file stream leak in `git history reword`. **Ready for `next`**.
- **`ad/gpg-strip-cr-before-lf` (Andrei Dziahel):** GPG/SSH signature parsing now strips CR only when it immediately precedes LF. **Needs review**.

---

### On the radar

- **`tc/replay-linearize`:** Toon Claes’s `--linearize` series is **ready for `next`** but may see a follow-up to address Junio’s behavioral note.
- **`hn/branch-delete-merged`:** "git branch --delete-merged <branch>" (Harald Nordgren) **needs review** for its opt-out configuration (`branch.<name>.deleteMerged`).
- **`ps/shift-root-in-graph`:** "git log --graph" indents parentless root commits (Patrick Steinhardt). **Peek-ahead approach may be dropped** in favor of a simpler solution.