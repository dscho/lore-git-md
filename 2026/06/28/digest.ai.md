# The Git Project Daily Digest
**2026/06/28 (UTC)**

## The day in brief
A **moderate-volume Sunday** (45 emails across 17 threads) saw **critical follow-ups** to recent regressions, **final readiness** for long-running series, and **new performance insights** into reftable’s quadratic-time behavior. The standout items: **Johannes Schindelin’s regression report** in `git replay --linearize` (v5) that silently drops commits, **Kristofer Karlsson’s merged `paint_down_to_common()` optimization** delivering 100-1000x speedups, and **Patrick Steinhardt’s reproducible scenario** for reftable’s quadratic-time bottleneck. **Harald Nordgren’s `git history squash` v6** and **Pablo Sabater’s GSoC `git cat-file --batch-command` v15** are now **technically complete** and ready for Junio’s final review.

---

## Notable threads

### Regression in `git replay --linearize` (v5) silently drops commits
**Thread**: [2026/06/08/18-37-18]
**Author**: Johannes Schindelin

Johannes Schindelin reported a **critical regression** in `git replay --linearize` (v5) that **silently drops commits** when replaying a single branch containing merge commits. The test case—`git replay --linearize --onto master~2 master~2..master`—incorrectly replays only the tip commit, omitting the first replayed commit ("Git 2.55-rc2"). Schindelin traced the issue to the removal of the `replayed_base` parameter, which was originally introduced to prevent this exact commit-dropping behavior. The regression affects the **predictable, all-or-nothing flattening** promised by `--linearize` and must be fixed before the next release.

**Key details**:
- **Root cause**: Removal of `replayed_base` in `pick_regular_commit()`.
- **Impact**: Silent commit dropping when replaying a single branch with merges.
- **Test case**: `master~2..master` with `--linearize --onto master~2`.
- **Files**: `replay.c`, `builtin/replay.c`.
- **Status**: **Urgent follow-up required**; Toon Claes is expected to send a patch restoring the parameter or redesigning the base-tracking logic.

---

### `paint_down_to_common()` optimization merged (100-1000x speedups)
**Thread**: [2026/06/20/10-36-53]
**Author**: Kristofer Karlsson

Kristofer Karlsson’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged** after addressing a regression in v3. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Benchmarks show **dramatic improvements**: `merge-base --all` across a 2.6M-commit monorepo drops from **3.67s to 5ms**, and step counts in git.git are reduced by **30-60%** for common queries.

**Key details**:
- **Optimization**: Early termination when `(!state->parent1_count || !state->parent2_count) && !state->mb_candidate_count`.
- **Regression fix**: Restored the `min_generation` guard to the generation-monotonicity BUG assertion.
- **Instrumentation**: Trace2 step counts via `trace2_data_intmax()`.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).
- **Files**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, test scripts.
- **Status**: **Merged**; no further changes expected.

---

### Reftable quadratic-time behavior: reproducible scenario identified
**Thread**: [2026/06/08/23-36-45]
**Author**: Patrick Steinhardt

Patrick Steinhardt identified a **reproducible scenario** for reftable’s quadratic-time behavior during bulk ref deletion/re-creation: **deleting and re-creating a large number of refs without compaction**. This workflow directly exercises the tombstone iteration bottleneck in `refs_verify_refname_available()` and `reftable_be_transaction_prepare()`, where both functions must scan all tombstones to confirm refname availability. Kristofer Karlsson’s **merged fix** (exposing tombstones to iterator bounds checks) reduces runtime from **O(n²) to O(n)** in this scenario, but its real-world impact remains unproven.

**Key details**:
- **Reproducible command**: `git for-each-ref | git update-ref --stdin`.
- **Bottleneck**: Tombstone iteration in `refs_verify_refname_available()` and `reftable_be_transaction_prepare()`.
- **Fix**: Kristofer’s patch exposes tombstones to iterator bounds checks.
- **Impact**: 8,000 refs: **1.8s → 0.145s** (100x speedup); 16,000 refs: **7.1s → 0.258s**.
- **Limitation**: The fix’s impact is negligible when compaction is enabled.
- **Status**: **Merged fix under reconsideration**; Patrick’s scenario may justify retaining it as a safeguard.

---

### `git history squash` v6: technically complete
**Thread**: [2026/06/14/17-52-17]
**Author**: Harald Nordgren

Harald Nordgren’s **v6 series** implementing `git history squash` is now **technically complete** and ready for Junio’s final review. The series folds a commit range into its oldest commit while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach. Key improvements in v6:
- **Input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`.
- **Multiple revision arguments**: Supports compound ranges (e.g., `@~3.. ^topic`).
- **`--reedit-message` template**: Aligns with `git rebase -i`’s squash behavior.
- **Ref-handling**: Rejects operations with interior refs by default, advising `--update-refs=head`.

**Key details**:
- **Files**: `builtin/history.c`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
- **Test coverage**: 747 lines, including range parsing, merge handling, ref safety, and template validation.
- **Status**: **Ready for final review**; no further design changes expected.

---

### `git cat-file --batch-command` v15: ready for merging
**Thread**: [2024/06/28/19-04-57]
**Author**: Pablo Sabater

Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries is **technically complete** and ready for merging. The series introduces a new `remote-object-info` command that lets clients request metadata (e.g., object size) for up to 10,000 objects in a single command, reducing network overhead. Key features:
- **Dynamic format validation**: Placeholders are validated based on server capabilities.
- **Security hardening**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.

**Key details**:
- **Files**: `builtin/cat-file.c`, `fetch-pack.c`, `connect.c`, `Documentation/git-cat-file.adoc`.
- **Open question**: Whether the client should fail explicitly or continue silently when metadata is missing.
- **Status**: **Ready for merging**; no further changes expected.

---

### `greplint.pl` regression: `test_grep`’s file check fails to catch bug
**Thread**: [2026/06/04/07-45-52]
**Author**: Junio C Hamano

Junio C Hamano identified a **critical oversight** in the `greplint.pl` series: `test_grep` already checks for file existence (`test -f "$last_arg"`), yet the bug in `t3420-rebase-autostash.sh` (where `file3` should not exist) persists even after conversion. The issue lies in the test’s logic—`file3` *should not exist* after `git rebase --quit`, so grepping it is incorrect regardless of the pattern. The `# lint-ok:` comment ("file may not exist after --quit") is a misstatement; it *never* exists, making the grep assertion fundamentally wrong.

**Key details**:
- **Root cause**: The linter’s conversion preserves the test’s incorrect assumption that `file3` *might* exist.
- **Fix**: Gábor’s 2021 patch (replacing `! grep dirty file3` with `test_path_is_missing file3`).
- **Risk**: `# lint-ok:` comments may hide other latent bugs.
- **Status**: **Follow-up required**; the series may need an audit of all `# lint-ok:` annotations.

---

## In brief

**`git branch --set-upstream-to` usability fix** -- Harald Nordgren’s **v3 series** improving error messages for misformatted remote/branch arguments (e.g., `origin main` instead of `origin/main`) is **merged into `next`**. The series adds two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and is ready for the next release.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**. The changes will be proposed for upstream Git in the coming weeks.

**CI hangs in `t5551`/`t5559`** -- Jeff King’s **3-patch series** increasing Apache’s `Timeout` directive to 600 seconds and isolating the expensive “many-tags” test case is **ready for merging**. The series resolves macOS CI hangs caused by Apache bug 70131.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true` or if the knob should be deprecated.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **2-line patch** fixing a regression in `set_curl_proxy_type()` (which incorrectly rejected HTTPS proxy URLs) is **merged into `master`**. The fix restores the intended behavior while preserving the hardening intent of the original commit.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue.
- **Reftable quadratic-time behavior**: Patrick Steinhardt’s reproducible scenario may justify retaining Kristofer’s fix as a safeguard.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **`git history squash`**: Harald Nordgren’s **v6 series** awaits Junio’s final review.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.