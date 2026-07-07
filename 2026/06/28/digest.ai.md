# The Git Project Daily Digest
**2026/06/28 (UTC)**

---

## The day in brief
A **moderate-volume Sunday** (45 emails across 17 threads) saw **critical regressions addressed**, **performance optimizations finalized**, and **long-running series reach completion**. The standout developments: **`git replay --linearize`’s post-merge regression** was diagnosed, **Tian Yuchen’s `paint_down_to_common()` optimization** received final approval, and **Harald Nordgren’s `git history squash`** series reached feature-complete status. A quiet but technically dense day, with **no single thread dominating**—instead, a steady stream of incremental progress.

---

## Notable threads

### `git replay --linearize` regression: silent commit dropping
**Johannes Schindelin** identified a **critical regression** in `git replay --linearize` (merged in v5) where the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was originally introduced to prevent this exact behavior. Schindelin’s test case—replaying `master~2..master` with `--linearize --onto master~2`—demonstrates the problem: only the tip commit is replayed, omitting the first commit in the range ("Git 2.55-rc2").

The regression is **urgent** and must be fixed before the next release. **Toon Claes** (the series author) has not yet responded, but the fix is expected to be straightforward: restore the `replayed_base` logic or redesign how multi-branch histories are handled. The thread highlights the **fragility of merge-linearization logic** and the need for more robust test coverage of edge cases.

**Key technical details**:
- **Files**: `replay.c` (core logic), `t3650-replay-basics.sh` (test coverage).
- **Root cause**: Removal of `replayed_base` parameter in `pick_regular_commit()`.
- **Test case**: `git replay --linearize --onto master~2 master~2..master`.
- **Expected behavior**: All commits in the range should be replayed.
- **Actual behavior**: Only the tip commit is replayed.

---

### `paint_down_to_common()` optimization: final approval
**Derrick Stolee** gave **final approval** to Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). The optimization delivers **100-1000x speedups** for asymmetric queries, such as `git merge-base --all` across a 2.6M-commit monorepo (3.67s → 5ms).

The series is now **fully reviewed and ready for merging**, with all feedback from v3 incorporated. Key improvements since v3:
- **Regression fix**: Restored the `min_generation` guard to the generation-monotonicity BUG assertion (patch 7/8).
- **Code clarity**: Unified halt conditions in the `paint_state` struct.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).

The optimization is a **major win for `git merge-base` performance** in asymmetric histories, with no known regressions. The series is **low-risk** (internal to the commit-reach subsystem) and **high-reward** (dramatic speedups for common asymmetric queries).

**Key technical details**:
- **Files**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, test scripts (`t6099-merge-base-side-exhaustion.sh`, `t6600-test-reach.sh`).
- **New symbols**: `paint_state` struct (with counters `parent1_count`, `parent2_count`, `mb_candidate_count`, `min_generation`, `last_gen`), `paint_count_update()`, `paint_queue_put()`, `paint_queue_get()`.
- **Behavior change**: Early termination when `(!state->parent1_count || !state->parent2_count) && !state->mb_candidate_count && last_gen < GENERATION_NUMBER_INFINITY`.
- **Test coverage**: Edge cases (self, duplicate-twos, pending-stale, infinity-both-sides), deterministic step-count assertions for all commit-graph modes.

---

### `git history squash`: feature-complete v6
**Harald Nordgren** posted **v6** of the `git history squash` series, which folds a commit range into its oldest commit while preserving descendant history. The series is now **feature-complete**, addressing all prior feedback:
- **Input validation**: Supports multiple revision arguments (e.g., `@~3.. ^topic`), rejects single-commit ranges, and detects refs pointing to commits inside the squashed range.
- **Merge commit handling**: Rejects merges with external parents but allows fully contained merges.
- **`--reedit-message`**: Seeds the editor with all folded-in commit messages, oldest first, mirroring `git rebase -i`’s squash behavior.
- **Ref safety**: Rejects operations with interior refs by default, advising users to use `--update-refs=head`.

The series is **ready for Junio’s final review**, with no further design changes expected. The implementation is **well-tested** (747-line test suite) and **low-risk** (internal to the `git history` command). The only open question is whether `--reedit-message` (or `--edit`) should be the default, but this is not a blocker.

**Key technical details**:
- **Files**: `builtin/history.c`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
- **New helpers**: `resolve_squash_range()`, `reject_fixupish_oldest()`, `find_interior_ref()`, `build_squash_message()`.
- **Behavior**: Folds a commit range into its oldest commit via a single three-way merge, then replays descendants.
- **Test coverage**: Range parsing, merge commit handling, ref safety, `--reedit-message` template construction, and `--update-refs` behavior.

---

### `greplint.pl`: fragile test fix and automation risks
**Junio C Hamano** and **Michael Montalbo** debated the risks of **automated test conversion** in the `greplint.pl` series, which replaces bare `grep` assertions with `test_grep`. The discussion centered on a **latent bug** in `t3420-rebase-autostash.sh`, where `! grep dirty file3` was incorrectly used to verify the absence of dirty content after `git rebase --quit`. The test should instead assert that `file3` does not exist at all (`test_path_is_missing file3`).

The thread exposed a **tension between automation and diagnostic rigor**: while `greplint.pl` aims to improve debuggability, its `# lint-ok:` exemptions can **preserve or mask pre-existing bugs**. Junio’s follow-up identified that `test_grep`’s existing file-existence check (`test -f "$last_arg"`) was bypassed by the linter’s exemption, preserving the bug. The discussion now focuses on whether the linter’s logic should be adjusted to **flag assertions on files that should not exist** (e.g., by cross-referencing `test_path_is_missing` calls).

**Key technical details**:
- **Files**: `t/test-lib-functions.sh` (`test_grep`), `t3420-rebase-autostash.sh` (failing test), `greplint.pl` (linter tool).
- **Root cause**: `# lint-ok:` comment bypassed `test_grep`’s file-existence check.
- **Fix**: Replace `! grep dirty file3` with `test_path_is_missing file3`.
- **Open question**: Should `greplint.pl`’s conversion logic be adjusted to avoid masking similar bugs?

---

### `git cat-file --batch-command`: final readiness
**Karthik Nayak** and **Pablo Sabater** clarified the **protocol extensibility** for Pablo’s GSoC series implementing `git cat-file --batch-command` for remote object metadata queries. The series is now **technically complete** and ready for merging, with all feedback addressed:
- **Dynamic format validation**: Placeholders are validated based on server capabilities.
- **Security hardening**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.

The only unresolved debate is **philosophical**: whether the client should fail explicitly or continue silently when metadata is missing. **No consensus has been reached**, but the behavior is now clearly documented: the client continues silently, matching local `git cat-file --batch-command` behavior.

**Key technical details**:
- **Files**: `builtin/cat-file.c`, `fetch-pack.c`, `connect.c`, `Documentation/git-cat-file.adoc`, `t1017-cat-file-remote-object-info.sh`.
- **New command**: `remote-object-info` (supports `%(objectname)` and `%(objectsize)` placeholders).
- **Behavior**: Batches requests to minimize remote communication overhead, with a 10,000-object limit for security.

---

### CI hangs in `t5551`/`t5559`: Apache timeout fix
**Jeff King (Peff)** posted a **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` caused by Apache bug 70131. The series:
1. **Increases Apache’s `Timeout` directive** from 300 to 600 seconds (patch 1/3).
2. **Isolates the expensive "many-tags" test case** into a dedicated repository (patch 2/3).
3. **Packs refs after creating many tags** to reduce `ls-refs` advertisement time (patch 3/3).

The series is **ready for merging**, with the first patch alone resolving the CI hangs. Peff and Junio are **ambivalent about the third patch’s necessity**, as the other patches already mitigate the timeout risk. The series is **low-risk** and **well-motivated**, addressing a persistent CI flake at its source.

**Key technical details**:
- **Files**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`.
- **Root cause**: Apache’s 300-second CGI timeout being hit during `ls-refs` advertisements of 100,000 loose refs.
- **Failure modes**: `curl 18` mid-transfer abort (HTTP/1.1) or indefinite hang (HTTP/2).

---

## In brief

**`git branch --set-upstream-to` usability fix** -- **Harald Nordgren**’s **v3 series** improving error messages for common command-line mistakes (e.g., `git branch --set-upstream-to=origin main`) is **merged into `next`**. The series adds two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and is **ready for `master`**.

**`gitk`/`git-gui` quiet builds** -- **Junio C Hamano** confirmed that **Johannes Sixt**’s patches aligning translation catalog generation with core Git’s quiet build conventions will be proposed for upstream Git inclusion in the coming weeks. The changes are **merged into Sixt’s testing branches**.

**`USE_NSEC` Meson parity** -- **Jeff King (Peff)**’s testing demonstrated that `USE_NSEC` is **safe on modern Linux** (ext4, ext2, vfat, CIFS, NTFS, FUSE), obsoleting historical "racy Git" concerns. The discussion now centers on whether the default should flip to `true` or if the knob should be deprecated.

**Reftable quadratic-time fix** -- **Kristofer Karlsson**’s patch fixing a quadratic-time scalability issue in the reftable backend is **under reconsideration** due to **limited real-world impact**. The patch exposes tombstones to iterator bounds checks, reducing runtime from O(n²) to O(n) for bulk ref deletion/re-creation workflows.

**French translation update** -- **Jiang Xin**’s pull request for Git 2.55.0 updates the `.po` files for 11 languages, including a mass typo-fix pass for French. The request is **ready for merging**.

**HTTPS proxy regression fix** -- **Johannes Schindelin**’s patch fixing a regression in HTTPS proxy URL handling is **merged into `master`**. The fix is a **two-line change** in `http.c` and restores the intended behavior.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **`git history squash`**: Harald Nordgren’s **v6 series** awaits Junio’s final review.
- **`excludes_file` migration**: Tian Yuchen’s series is ready for `next` after resolving the guardrail debate.
- **`--track=fetch` debate**: Harald Nordgren’s feature remains stalled on workflow concerns.