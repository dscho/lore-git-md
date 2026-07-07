# The Git Project Daily Digest
**2026/06/28 (Sunday) – A Week of Hardening, Optimizations, and Urgent Fixes**

## The day in brief
A **moderate-volume Sunday** (45 emails, 17 threads) saw **critical regressions addressed**, **performance optimizations finalized**, and **long-running series reach readiness**. The standout developments: **`git replay --linearize`’s regression fix** surfaced, **Tian Yuchen’s `paint_down_to_common()` optimization** received final approval, and **Harald Nordgren’s `git history squash`** advanced to v7. A quiet day for controversy—most threads focused on **polish and integration**.

---

## Notable threads

### `git replay --linearize` regression fix surfaces
**Thread: [PATCH 0/3] replay: introduce --linearize option**

Johannes Schindelin identified a **critical regression** in `git replay --linearize` (merged in v5): the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was necessary to maintain the correct parent for commits following a merge. Schindelin’s test case—replaying `master~2..master` with `--linearize --onto master~2`—demonstrates the bug: only the tip commit is replayed, omitting the first commit ("Git 2.55-rc2").

**Key details**:
- **Root cause**: Loss of `replayed_base` tracking in `pick_regular_commit()`.
- **Impact**: Silent data loss in linearization workflows.
- **Fix direction**: Restore `replayed_base` or redesign multi-branch handling.
- **Files touched**: `replay.c`, `t3650-replay-basics.sh`.

**Status**: **Urgent follow-up needed**. Toon Claes is expected to send a patch to restore `replayed_base` or propose an alternative. The regression must be fixed before the next release.

---

### `paint_down_to_common()` optimization receives final approval
**Thread: [PATCH v4 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories**

Derrick Stolee **approved the entire 8-patch series**, marking Tian Yuchen’s optimization of `paint_down_to_common()` as **ready to merge**. The series terminates merge-base walks early when one side’s commit queue is exhausted, yielding **100-1000x speedups** for asymmetric histories (e.g., repositories with import grafts). Key improvements since v3:
- **Regression fix**: Restored the `min_generation` guard to the BUG assertion in patch 7/8.
- **Code clarity**: Unified halt conditions in `struct paint_state`.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).

**Key details**:
- **Performance**: `merge-base --all` across a 2.6M-commit monorepo drops from **3.67s to 5ms**.
- **Test coverage**: Deterministic step-count assertions for all commit-graph modes (none, full, half, no-gdat).
- **Files touched**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, test scripts.

**Status**: **Merged into `next`**. No further changes expected.

---

### `git history squash` advances to v7
**Thread: [PATCH v6 0/4] replay: introduce `git history squash`**

Harald Nordgren posted **v7 of `git history squash`**, addressing all prior feedback and adding **stricter input validation**. The series folds a commit range into its oldest commit while preserving descendant history, offering a simpler alternative to `git rebase -i` for linearization. Key changes in v7:
- **Reject ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`** (since the marker’s target cannot lie inside the range).
- **Adopt `git rebase -i`’s squash-message template** for `--reedit-message`.
- **Reorder CLI usage** to place dashed options before `<revision-range>`.
- **Expand documentation** to clarify merge limitations and feature overview.

**Key details**:
- **Behavior**: Rejects operations with interior refs by default, advising `--update-refs=head`.
- **Test coverage**: 747-line test suite covering range parsing, merge handling, and ref safety.
- **Files touched**: `builtin/history.c`, `sequencer.c`/`sequencer.h`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`.

**Status**: **Ready for Junio’s final review**. The series is **feature-complete** and addresses all prior objections.

---

### `greplint.pl` series hits design snag
**Thread: [PATCH 0/6] Introduce `greplint.pl` to convert `grep` assertions to `test_grep`**

Junio C Hamano identified a **critical oversight** in the `greplint.pl` series: the existing `test_grep` implementation already checks for file existence (`test -f "$last_arg"`), yet the bug in `t3420-rebase-autostash.sh` (where `file3` should not exist) persists even after conversion. The issue lies in how the test’s logic interacts with `test_grep`’s behavior, not the file-checking itself. Gábor SZEDER’s 2021 patch (replacing `! grep dirty file3` with `test_path_is_missing file3`) remains the correct fix.

**Key details**:
- **Root cause**: The linter’s `# lint-ok:` exemption preserves the test’s incorrect assumption that `file3` *might* exist.
- **Risk**: `# lint-ok:` comments may mask other latent bugs.
- **Files touched**: `t/test-lib-functions.sh`, `t3420-rebase-autostash.sh`, `greplint.pl`.

**Status**: **Design debate ongoing**. Junio suggests auditing all `# lint-ok:` comments to ensure they don’t mask similar issues. The series’ future hinges on whether the linter’s automation can be reconciled with diagnostic rigor.

---

### `git cat-file --batch-command` finalizes protocol design
**Thread: [PATCH v14 0/13] cat-file: add --batch-command remote-object-info**

Karthik Nayak and Pablo Sabater **finalized the wire protocol design** for `git cat-file --batch-command`’s `remote-object-info` command. Future metadata features (e.g., `objecttype`) will be appended to the `object-info` capability value (e.g., `object-info=size type`). The documentation in `gitprotocol-v2.adoc` now specifies:
- Full object IDs are required (no short OIDs).
- Servers omit all attributes for unrecognized object IDs (returning only the object ID).
- The `obj-size` field and response format are explicitly documented.

**Key details**:
- **Extensibility**: Space-separated attribute list format (e.g., `size type`).
- **Security**: Strict protocol v2 enforcement and input validation.
- **Files touched**: `gitprotocol-v2.adoc`.

**Status**: **Ready for merging**. The series is **technically complete** and addresses all prior feedback.

---

### CI hangs in `t5551`/`t5559` resolved
**Thread: [PATCH 0/3] macOS CI hang in t5551/t5559 – root cause and fix**

Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` was **merged**. The series:
1. **Increases Apache’s `Timeout` directive** from 300 to 600 seconds.
2. **Isolates the expensive "many-tags" test case** into a dedicated repository.
3. **Packs refs after creating many tags** to reduce `ls-refs` advertisement time.

**Key details**:
- **Root cause**: Apache bug 70131 causing HTTP/2 stalls or `curl 18` mid-transfer aborts.
- **Files touched**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`.

**Status**: **Merged**. The series resolves the CI flakes with minimal risk.

---

## In brief

**`git branch --set-upstream-to` usability fix** -- Harald Nordgren’s **v3 series** improving error messages for missing slashes (e.g., `origin main` → `origin/main`) was **merged into `next`**. The series adds two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and includes thorough test coverage.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions was **merged (gitk) or superseded (git-gui)**. The changes ensure `make -s` produces no output during catalog generation.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option was **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**Reftable memory leak fix** -- Jeff King’s **bugfix patch** for a memory leak in `reftable_writer_new` was **merged**. The fix moves the allocation after input validation, plugging a leak introduced in February 2025.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **bugfix patch** correcting a regression in HTTPS proxy URL handling was **merged into `master`**. The fix adds a `return 0` for HTTPS proxies in `set_curl_proxy_type()`.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to restore `replayed_base` or redesign multi-branch handling.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **`git history squash`**: Harald Nordgren’s **v7 series** awaits Junio’s final review.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.
- **`excludes_file` migration**: Tian Yuchen’s series is ready for `next` after resolving the guardrail debate.