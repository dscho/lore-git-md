# The Git Project Daily Digest
**2026/06/28 (Sunday) – A day of regressions, resolutions, and refactoring**

Today’s traffic was **lighter than average** (45 emails across 17 threads) but **technically dense**, with a mix of **regression fixes, finalized features, and architectural refactoring**. The standout developments: a **post-merge regression in `git replay --linearize`** dominated discussion, while **Tian Yuchen’s `paint_down_to_common()` optimization** and **Harald Nordgren’s `git history squash`** reached key milestones. The day also saw **CI fixes land**, **build system debates**, and **last-minute l10n updates** ahead of Git 2.55.0.

---

## Notable threads

### `git replay --linearize` regression exposes merge-commit handling flaw
Johannes Schindelin identified a **critical regression** in the recently merged `git replay --linearize` feature: when replaying a single branch containing merge commits (e.g., `master~2..master`), the command **silently drops commits** by replaying the tip directly onto the `--onto` target. The issue stems from the removal of the `replayed_base` parameter in v5, which was originally introduced to prevent this exact behavior. Schindelin’s test case (`replaying master~2..master --linearize --onto master~2`) reveals the flaw: the tip commit is replayed directly onto `master~2`, omitting the first replayed commit ("Git 2.55-rc2").

The regression is **urgent**—it affects a feature merged just two weeks ago—and highlights a **design tension** in how `--linearize` should handle multi-branch histories. Schindelin’s critique is precise: the current approach (cherry-picking each commit onto its rewritten parent) differs from `git rebase --no-rebase-merges`, which replays all commits in a linear sequence. The thread is now focused on whether to **restore the `replayed_base` logic** or redesign the algorithm to handle merge commits more explicitly. Toon Claes is expected to send a follow-up patch early next week.

**Key files**: `replay.c`, `t/t3650-replay-basics.sh`.
**Status**: Regression confirmed; fix pending.

---

### `paint_down_to_common()` optimization lands after regression fix
Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged** today. The series delivers **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts) by terminating early when one side’s commit queue is exhausted. The final hurdle—a **regression in patch 7/8** caused by an unconditional BUG assertion—was resolved in v4, which restored the `min_generation` guard to the generation-monotonicity check.

The series is a **major win for performance** and includes:
- **Trace2 instrumentation** to measure commits walked (patch 4/8).
- **Unified halt conditions** in `struct paint_state` (patch 5/8).
- **New technical documentation** (`paint-down-to-common.adoc`, patch 1/8).
- **Edge-case test coverage** (patches 2/8 and 3/8).

Derrick Stolee’s **unqualified approval** ("LGTM") for the final patch removes the last technical obstacle. The optimization is **internal-only** (no user-facing changes) and has no known regressions.

**Key files**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`.
**Status**: Merged to `master`.

---

### `git history squash` v6 addresses input flexibility and validation
Harald Nordgren’s **4-patch series** implementing `git history squash` reached **feature-complete status** in v6. The command folds a commit range into its oldest commit while preserving descendant history, offering a faster alternative to `git rebase -i` for linearizing histories. The latest iteration addresses two key critiques:
1. **Input flexibility**: The series now supports **multiple revision arguments** (e.g., `@~3.. ^topic`), allowing users to exclude commits already on another branch. This directly addresses Junio C Hamano’s usability concern about the single-argument design.
2. **Stricter validation**: Single-commit ranges (e.g., `@^!`) and non-ancestor ranges are now explicitly rejected.

The series also includes:
- **`--reedit-message`** to combine all folded-in commit messages (oldest first).
- **Ref safety checks**: Rejects operations where refs point to commits inside the squashed range, with an advice message suggesting `--update-refs=head`.
- **Test coverage**: 550+ lines of tests covering compound ranges, merge topologies, and edge cases.

The implementation is **clean and well-documented**, with no open technical questions. Junio’s final review is pending, but the series is **likely to merge soon**.

**Key files**: `builtin/history.c`, `t/t3455-history-squash.sh`.
**Status**: Ready for final review.

---

### CI hangs in `t5551`/`t5559` resolved with Apache timeout increase
Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` was **merged today**. The series targets Apache bug 70131, which causes HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. The core fix—**increasing Apache’s `Timeout` directive from 300 to 600 seconds**—directly addresses the root cause, while the other two patches optimize the test suite to reduce timeout risk:
1. **Patch 1/3**: Raises `Timeout` in `t/lib-httpd/apache.conf`.
2. **Patch 2/3**: Isolates the expensive "many-tags" test case into a dedicated repository.
3. **Patch 3/3**: Packs refs after creating 100,000 tags, reducing `ls-refs` time from ~1.2s to 24ms.

Junio C Hamano and Peff debated the necessity of the third patch (ref-packing), ultimately deciding to **merge it as a belt-and-braces measure**. The series is **low-risk** and resolves a persistent CI flake.

**Key files**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`.
**Status**: Merged to `master`.

---

### `USE_NSEC` Meson option sparks platform compatibility debate
D. Ben Knoble’s patch adding a `nanosec` Meson option to mirror Autotools’ `USE_NSEC` sparked a **surprising debate** about the knob’s viability. Jeff King’s testing revealed that modern Linux filesystems (ext4, ext2, vfat, CIFS) **preserve nanosecond timestamps correctly**, contradicting the long-standing assumption that `USE_NSEC` is "subtly broken" on network filesystems. However, Peff also surfaced a **new interoperability risk**: mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit) could cause performance degradation, as millisecond-resolution timestamps would appear "stat-dirty" to nanosecond-aware Git.

The discussion now centers on whether the knob should:
1. **Flip to `true` by default** (Peff’s preference, given the Linux findings).
2. **Be deprecated** (Patrick Steinhardt’s position, arguing it adds no value).
3. **Remain as-is** (current status quo).

No patch changes are proposed yet, but the thread has **shifted from build system parity to platform compatibility**. The original patch (adding the Meson option) remains valid, but its justification is now in question.

**Key files**: `meson.build`, `meson_options.txt`.
**Status**: Debate ongoing; no action required yet.

---

## In brief

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` was **merged after resolving a guardrail debate**. The series aligns with the `the_repository` removal effort and is **ready for `next`**.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **2-line fix** for a regression rejecting HTTPS proxy URLs was **merged**. The patch corrects a control-flow error in `set_curl_proxy_type()` introduced in commit `663d7abe07ea`.

**Reftable memory leak fix** -- Jeff King’s **4-line fix** for a memory leak in `reftable/writer.c` was **merged**. The leak was introduced in February 2025 and recently flagged by Coverity.

**French translation update** -- Jean-Noël Avila’s **mass typo-fix pass** for the French `.po` file was **merged as part of a 11-language l10n update** for Git 2.55.0.

**`git branch --set-upstream-to` usability fix** -- Harald Nordgren’s **v3 series** improving error messages for missing-slash arguments (e.g., `origin main` instead of `origin/main`) was **merged**. The series adds a new config option (`advice.setUpstreamFailure`) and is **ready for the next release**.

---

## On the radar

**`git repack --geometric --cruft`** -- Taylor Blau’s **10-patch RFC** combining geometric and cruft repack modes faces a **correctness issue** in its reachability filtering. Junio identified that the two-phase traversal may retain unreachable tags and objects, undermining the separation of reachable/unreachable objects. A fix is needed before merging.

**ODB abstraction stack** -- Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use a `source` field (replacing the `whence` enum) is **conceptually approved** and awaits substantive review. The series is foundational for pluggable ODB backends.

**`git history --reword` leak fix** -- Junio’s **bugfix patch** addressing a file stream leak in `git history --reword` is **merge-ready**. The fix is critical for Windows compatibility.