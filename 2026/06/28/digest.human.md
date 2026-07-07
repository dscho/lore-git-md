Here’s the daily digest for **2026/06/28** (UTC), covering the Git mailing list’s key developments in a concise, front-page-style overview:

---

# The day in brief
**2026/06/28** was a **heavy but focused** day on the Git mailing list, with **45 emails across 17 threads**. The standout themes: **performance optimizations landed**, **security hardening merged**, and **regressions surfaced** in recently integrated features. The most critical thread? A **post-merge regression in `git replay --linearize`** that silently drops commits—a must-fix before the next release. Other highlights: **Tian Yuchen’s `paint_down_to_common()` optimization** (100-1000x speedups) was fully approved, and **Patrick Steinhardt’s reftable security series** (11 patches) graduated to `next`.

---

## Notable threads

### `git replay --linearize` regression: silent commit dropping
**Headline**: A **post-merge regression** in `git replay --linearize` (v6) was reported by Johannes Schindelin, where the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was critical for tracking the correct parent during linearization. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign the multi-branch handling logic. **Urgency**: High—this must be fixed before the next release.

### Key details

- **Root cause**: Removal of `replayed_base` in `pick_regular_commit()`.
- **Test case**: Replaying `master~2..master` with `--linearize --onto master~2` drops the first commit ("Git 2.55-rc2").
- **Files touched**: `replay.c`, `builtin/replay.c`.
- **Status**: Regression confirmed; fix pending.

---

### `paint_down_to_common()` optimization: 100-1000x speedups
**Headline**: Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged**. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Benchmarks show **dramatic improvements**: `merge-base --all` across a 2.6M-commit monorepo drops from **3.67s to 5ms**, and step counts in git.git are reduced by **30-60%**.

### Key details

- **Core change**: Early termination when `(!state->parent1_count || !state->parent2_count) && !state->mb_candidate_count`.
- **Regression fix**: Restored the `min_generation` guard to the generation-monotonicity BUG assertion (patch 7/8).
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).
- **Files touched**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, test scripts.
- **Status**: Merged; no known regressions.

---

### Reftable security hardening: fuzzing and fixes
**Headline**: Patrick Steinhardt’s **11-patch series** hardening Git’s reftable backend against corrupted files was **merged into `next`**. The series fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in the reftable parser, and introduces a **libFuzzer-based fuzzer** to prevent regressions. The fuzzer now runs for **2+ hours without finding new issues**, setting a new bar for security testing in Git.

### Key details

- **Critical fixes**:
  - Patch 5/11: Heap-buffer-overflow in log block reconstruction.
  - Patch 11/11: OOB read on truncated tables.
- **Fuzzing**: Added libFuzzer infrastructure and CI integration (patches 1–2).
- **Files touched**: `reftable/`, `t/`, CI scripts.
- **Status**: Merged; internal-only (no user-facing changes).

---

### `git cat-file --batch-command`: final readiness
**Headline**: Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries is **technically complete** and ready for merging. The series introduces a new `remote-object-info` command that lets clients request metadata (e.g., object size) for up to 10,000 objects in a single command, reducing network overhead for partial clone workflows.

### Key details

- **Dynamic format validation**: Placeholders are validated based on server capabilities.
- **Security**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.
- **Open debate**: Silent failures vs. explicit errors (no consensus yet).
- **Files touched**: `builtin/cat-file.c`, `fetch-pack.c`, `Documentation/`.
- **Status**: Ready for Junio’s final review.

---

### `git history squash`: input validation and usability
**Headline**: Harald Nordgren’s **v6 series** for `git history squash` (folding commit ranges into a single commit) is **functionally complete** and addresses all prior feedback. The series now supports **multiple revision arguments** (e.g., `@~3.. ^topic`), stricter input validation (rejecting ranges whose oldest commit is a `fixup!`/`squash!`), and a `--reedit-message` template aligned with `git rebase -i`.

### Key details

- **Input validation**: Rejects single-commit ranges, empty ranges, and ranges with multiple bases.
- **Merge handling**: Rejects merges with external parents but allows fully contained merges.
- **Ref safety**: Rejects operations with interior refs by default, advising `--update-refs=head`.
- **Files touched**: `builtin/history.c`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
- **Status**: Ready for Junio’s final review.

---

### CI hangs in `t5551`/`t5559`: Apache timeout fix
**Headline**: Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` is **ready for merging**. The series increases Apache’s `Timeout` directive from 300 to 600 seconds and isolates the expensive "many-tags" test case into a dedicated repository. The root cause was **Apache bug 70131**, which triggered HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs.

### Key details

- **Patch 1/3**: Increases `Timeout` to 600 seconds in `t/lib-httpd/apache.conf`.
- **Patch 2/3**: Isolates the "many-tags" test case into `many-tags.git`.
- **Patch 3/3**: Packs refs after creation to reduce `ls-refs` advertisement time (marginal benefit).
- **Files touched**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`.
- **Status**: Ready for merging; patch 3/3 may be dropped as redundant.

---

## In brief

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

**HTTPS proxy regression** -- Johannes Schindelin’s **2-line fix** for a regression rejecting HTTPS proxy URLs was **merged into `master`**. The fix restores the intended behavior of accepting HTTPS proxies, which the code already supports.

**Reftable memory leak** -- Jeff King’s **1-line fix** for a memory leak in `reftable_writer_new` was **merged into `pu`**. The fix moves the allocation after input validation.

---

## On the radar
- **`git repack --geometric --cruft`**: Taylor Blau’s **10-patch RFC** faces a correctness issue in its reachability filtering. A fix is needed before merging.
- **`git branch --delete-merged`**: Harald Nordgren’s **18-iteration series** is feature-complete and ready for Junio’s final assessment.
- **ODB abstraction**: Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` awaits substantive review.
- **`git log -L` improvements**: Karthik Nayak’s **7-patch series** extending `git log -L` to support range-scoped `--stat`, `--check`, and `-G` is technically complete.

---

## The day’s texture
Traffic was **heavier than average**, with a **technical focus** on performance, security, and regressions. The **`git replay --linearize` regression** dominated late-day discussion, overshadowing the **merging of Tian Yuchen’s optimization** and **Patrick Steinhardt’s security series**. The **GSoC `git cat-file --batch-command` series** reached a milestone, while **Harald Nordgren’s `git history squash`** and **Jeff King’s CI fixes** rounded out the day’s progress. The tone was **collaborative but urgent**, with regressions and performance wins driving the conversation.