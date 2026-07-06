## The day in brief

Sunday, June 28, 2026 brought **45 emails across 17 threads**, a **light but technically dense** day. The standout developments: **Harald Nordgren’s `git history squash` v6 series** landed with support for multiple revision arguments, **Tian Yuchen’s `paint_down_to_common()` optimization** was fully approved after fixing a regression, and **Jeff King’s CI timeout fix** for macOS runners reached final form. A **post-merge regression in `git replay --linearize`** resurfaced, requiring urgent follow-up.

---

## Notable threads

### `git history squash` v6 lands with multiple-revision support
Harald Nordgren’s **four-patch series** implementing `git history squash`—a new subcommand to fold a commit range into its oldest commit while preserving descendant history—reached **v6** and is now **feature-complete**. The headline change: the command now supports **multiple revision arguments** (e.g., `@~3.. ^topic`), addressing Junio C Hamano’s usability concern about the earlier single-argument limitation. The range resolution logic has been rewritten to treat any boundary commit other than the oldest in-range commit’s parent as evidence of multiple bases, ensuring unambiguous behavior.

The series is **well-tested**, with 550+ lines of new tests covering compound ranges, merge topologies (including external-parent rejection), and interior ref detection. The ref-handling logic remains strict: the command rejects operations where any ref points to a commit inside the squashed range, advising users to use `--update-refs=head` to retarget only the current branch. The `--reedit-message` flag gathers all folded-in commit messages (oldest first) into an editor template, mirroring `git rebase -i` behavior.

**Files touched**: `builtin/history.c`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, and a new test script `t/t3455-history-squash.sh`. The series is **ready for Junio’s final review**, with no further design changes expected.

---

### `paint_down_to_common()` optimization fully approved
Tian Yuchen’s **eight-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged** after fixing a **critical regression** in v3. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). The optimization yields **100-1000x speedups** for asymmetric queries, with benchmarks showing step-count reductions from 72,264 to 44,599 for `merge-base --all v2.0.0 v2.55.0-rc1`.

The v4 iteration **fixed the regression** by restoring the `min_generation` guard to the generation-monotonicity BUG assertion, ensuring correctness when the threshold is not set. The series also includes **trace2 instrumentation** (patch 4/8) to measure commits walked, **code clarity improvements** (patches 5–6/8), and a **new technical document** (`paint-down-to-common.adoc`) explaining the algorithm and termination conditions. Derrick Stolee’s **unqualified approval** for the entire series removes the last open technical question.

**Files touched**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, and test scripts (`t6099-merge-base-side-exhaustion.sh`, `t6600-test-reach.sh`). The series is **low-risk** (internal to the commit-reach subsystem) and **high-reward** (dramatic speedups for common asymmetric queries).

---

### CI timeout fix for macOS runners reaches final form
Jeff King’s **three-patch series** addressing macOS CI hangs in `t5551` and `t5559` reached its **final form**, implementing the consensus solution to increase Apache’s `Timeout` directive from 300 to 600 seconds. The series directly targets **Apache bug 70131**, which causes HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. The first patch (`t/lib-httpd: bump apache timeout`) is the core fix, while the second and third patches are optimizations: isolating the expensive "many-tags" test case and packing refs to reduce advertisement time.

Junio C Hamano’s **surface-level review** of the second patch noted a minor stylistic nit (non-bare repository initialization), but the series is otherwise **uncontroversial and ready to merge**. The timeout increase is conservative (600 seconds is double the default but far below Michael Montalbo’s tested 3600-second value), and the optimizations reduce the risk of hitting the timeout in the first place.

**Files touched**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`. The series is a **well-motivated, low-risk fix** for a persistent CI flake.

---

### `git replay --linearize` regression resurfaces
Johannes Schindelin reported a **post-merge regression** in `git replay --linearize` (merged in v5): the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact commit-dropping behavior. Schindelin’s test case (replaying `master~2..master` with `--linearize --onto master~2`) demonstrates the problem: the tip commit is incorrectly replayed directly onto the `--onto` target, omitting the first replayed commit ("Git 2.55-rc2").

The regression is **urgent** and must be fixed before the next release. Toon Claes is expected to send a follow-up patch to restore the `replayed_base` logic or redesign how multi-branch histories are handled. The thread highlights the complexity of `--linearize` when handling three branches where one is a merge of the other two, and Schindelin warns that attempts to preserve non-reachability between branches would likely surprise users.

**Files touched**: `replay.c` (likely). The fix is **blocking for the next release**.

---

### `excludes_file` libification guardrail debate resolved
Tian Yuchen and Junio C Hamano resolved the **final design question** for the `excludes_file` migration series, part of the ongoing `the_repository` removal effort. The debate centered on whether the new `repo_excludes_file()` getter should use a silent early return or a `BUG()` assertion for uninitialized repositories. The consensus: **remove the redundant `repo->initialized` check** and rely entirely on `repo_config_values()` to enforce initialization. This aligns with the broader libification principle of centralizing validation logic in a single helper.

The patch is now **ready for `next`**, with the only remaining action being a one-line removal in `environment.c`. The stricter design surfaces previously hidden invalid calls (e.g., tests passing uninitialized repositories to the getter), which will need to be addressed in separate patches. The resolution sets a precedent for future repository-aware getters (e.g., `attributes_file` migration).

**Files touched**: `environment.c`, `repository.c`. The series is **foundational for libification**.

---

## In brief

**Reftable compaction fix** -- Jeff King fixed a memory leak in `reftable/writer.c` where the `reftable_writer` struct was allocated before input validation. The fix moves the allocation after the block-size check, plugging the leak and improving code hygiene. The issue was introduced in February 2025 and recently flagged by Coverity.

**French translation update** -- Jean-Noël Avila’s pull request for Git 2.55.0 updates the French `.po` file and includes a mass typo-fix pass. The request covers 11 languages and is **ready for merging**.

**HTTPS proxy regression fix** -- Junio C Hamano fast-tracked Johannes Schindelin’s fix for a regression in commit `663d7abe07ea`, which incorrectly rejected HTTPS proxy URLs. The fix is a two-line change in `http.c` and has been merged into `master`.

**`USE_NSEC` Meson parity discussion** -- Jeff King’s testing suggests the `USE_NSEC` knob (for sub-second timestamp tracking) is obsolete on modern Linux filesystems, but interoperability risks remain when mixing `USE_NSEC` and non-`USE_NSEC` implementations. The discussion now centers on whether the default should flip to `true`.

**`git branch --set-upstream-to` usability fix** -- Junio C Hamano approved the final patch in Harald Nordgren’s v3 series, which improves error messages when users misformat remote/branch arguments (e.g., `origin main` instead of `origin/main`). The series is **ready for merging**.

**`gitk`/`git-gui` quiet builds** -- Junio C Hamano confirmed the procedural status of Harald Nordgren’s series aligning translation catalog generation with core Git’s quiet build conventions. The patches are **merged (gitk) or superseded (git-gui)**.

---

## On the radar

**`git repack --geometric --cruft` RFC** -- Taylor Blau’s series combining `--geometric` and `--cruft` repack modes faces a **correctness issue** in its reachability filtering. Junio identified that the two-phase traversal may retain unreachable tags and objects, undermining the separation of reachable/unreachable objects. A fix is needed before merging.

**`git history --reword` file stream leak** -- Junio’s bugfix patch for a file stream leak in `git history --reword` is **merge-ready** and addresses a Windows-specific issue.

**ODB abstraction stack** -- Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use a `source` field awaits substantive review. The series is foundational for pluggable ODB backends.