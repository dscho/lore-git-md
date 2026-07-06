# The Git Project Daily Digest -- 2026/06/28 (Sunday)

## The day in brief
A **moderate-volume Sunday** (45 emails, 17 threads) saw **critical regressions addressed**, **performance optimizations finalized**, and **long-running series reach completion**. The standout developments: **Tian Yuchen’s `paint_down_to_common()` optimization** (8 patches) was **fully approved and merged** after fixing a regression, **Harald Nordgren’s `git history squash`** (4 patches) reached **feature-complete status** with stricter input validation, and **Jeff King’s CI timeout fix** (3 patches) resolved persistent macOS hangs. A **post-merge regression in `git replay --linearize`** emerged late in the day, requiring urgent follow-up.

---

## Notable threads

### `paint_down_to_common()` optimization fully approved and merged
Tian Yuchen’s **8-patch series** optimizing Git’s merge-base calculation for one-sided histories was **fully approved and merged** after addressing a critical regression in v3. The series teaches `paint_down_to_common()` to terminate early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts or shallow histories). The optimization delivers **100-1000x speedups** for asymmetric queries, with benchmarks showing step-count reductions from 72,264 to 44,599 for `merge-base --all v2.0.0 v2.55.0-rc1` in git.git.

The v4 iteration fixed the regression (an unconditional BUG assertion in patch 7/8) by restoring the `min_generation` guard to the generation-monotonicity check. The series also includes **trace2 instrumentation** (patch 4/8) to measure commits walked, **edge-case test coverage** (patches 2/8 and 3/8), and a **new technical document** (`paint-down-to-common.adoc`, patch 1/8) explaining the algorithm’s mechanics. Derrick Stolee **unconditionally approved** the series, calling the instrumentation "clean" and noting it "over-achieves" by capturing step counts for all commit-graph modes. The changes are **internal-only**, with no user-visible behavior changes.

**Files touched**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, test scripts (`t6099-merge-base-side-exhaustion.sh`, `t6600-test-reach.sh`).
**Key symbols**: `paint_state` struct (with counters `parent1_count`, `parent2_count`, `mb_candidate_count`), `paint_queue_get()`, trace2 step counter.

---

### `git history squash` reaches feature-complete status
Harald Nordgren’s **4-patch v6 series** implementing `git history squash`—a new subcommand to fold a commit range into its oldest commit while preserving descendant history—is now **feature-complete** and ready for final review. The series addresses all prior feedback, including **stricter input validation** (rejecting single-commit ranges and non-ancestor ranges) and **support for multiple revision arguments** (e.g., `@~3.. ^topic`), which directly resolves Junio C Hamano’s usability concern from earlier iterations.

The implementation uses a single three-way merge of the entire range against the oldest commit’s parent, followed by replaying descendants. The `--reedit-message` flag gathers all folded-in commit messages (oldest first) into an editor template, mirroring `git rebase -i` behavior. The series also introduces **ref safety checks**: the command rejects operations where any ref points to a commit inside the squashed range, with an advice message suggesting `--update-refs=head` to retarget only the current branch. Test coverage is comprehensive, spanning **compound ranges, merge topologies, and interior ref detection**.

**Files touched**: `builtin/history.c`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
**Key symbols**: `resolve_squash_range()`, `find_interior_ref()`, `build_squash_message()`, `--update-refs[=head]`.

---

### CI timeout fix resolves macOS hangs
Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` was **finalized and merged**. The series directly targets Apache bug 70131, which causes HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. The core fix (patch 1/3) increases Apache’s `Timeout` directive from 300 to 600 seconds in `t/lib-httpd/apache.conf`, addressing the root cause without client-side workarounds. Complementary optimizations (patches 2/3 and 3/3) isolate the expensive "many-tags" test case and pack refs to reduce advertisement time from ~1.2 seconds to 24 ms.

Junio C Hamano **merged the series** after a brief discussion about the necessity of the ref-packing optimization (patch 3/3), which was ultimately retained as a belt-and-braces measure. The changes are **test-suite-only**, with no user-visible impact.

**Files touched**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`.

---

### `git replay --linearize` regression requires follow-up
Johannes Schindelin reported a **post-merge regression** in `git replay --linearize` (merged in v5): the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent commit-dropping in multi-branch scenarios. Schindelin’s test case (replaying `master~2..master` with `--linearize --onto master~2`) demonstrates the bug: the tip commit is incorrectly replayed directly onto the `--onto` target, omitting the first replayed commit ("Git 2.55-rc2").

Toon Claes is expected to send a follow-up patch to restore the `replayed_base` logic or redesign how multi-branch histories are handled. The regression is **urgent**, as it affects a recently merged feature, and must be fixed before the next release.

**Files touched**: `replay.c` (expected in follow-up).
**Key symbols**: `pick_regular_commit()`, `replayed_base`.

---

### `excludes_file` libification guardrail resolved
Tian Yuchen and Junio C Hamano **resolved the final design question** in the `excludes_file` libification series, agreeing to **remove the redundant `repo->initialized` check** from `repo_excludes_file()` and rely entirely on `repo_config_values()` for initialization validation. The change aligns with the broader libification principle of centralizing validation logic in a single helper, and the patch is now **ready for `next`**.

The stricter design surfaced previously hidden invalid calls (e.g., tests passing uninitialized repositories to the getter), which will need to be addressed in separate patches. The resolution sets a precedent for future repository-aware getters (e.g., `attributes_file` migration).

**Files touched**: `environment.c`.
**Key symbols**: `repo_excludes_file()`, `repo_config_values()`.

---

### HTTPS proxy regression fix merged
Junio C Hamano **merged Johannes Schindelin’s fix** for a regression in HTTPS proxy URL validation. The patch adds a missing `return 0` in `set_curl_proxy_type()` (in `http.c`), ensuring HTTPS proxy URLs are correctly accepted. The regression was introduced in commit `663d7abe07ea` (http: reject unsupported proxy URL schemes, 2026-05-05) and caused Git to incorrectly reject HTTPS proxies despite the code already supporting them. The fix is **minimal and self-contained**, with no tests required.

---

## In brief

**Reftable memory leak fix** -- Jeff King fixed a memory leak in `reftable/writer.c` where the `reftable_writer` struct was allocated before input validation. The fix moves the allocation after the block-size check, plugging the leak and improving code flow. The issue was introduced in commit `445f9f4f35` (February 2025) and flagged by Coverity.

**`git branch --set-upstream-to` usability fix** -- Harald Nordgren’s **v3 series** improving error messages for misformatted remote/branch arguments (e.g., `git branch --set-upstream-to=origin main`) was **merged**. The series adds config-based advice (`advice.setUpstreamFailure`) and strict validation to guide users toward correct syntax. Junio approved the patches after minor design quibbles were resolved.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning gitk and git-gui’s translation catalog generation with core Git’s quiet build conventions was **merged**. The patches remove `--statistics` from `msgfmt` invocations and conditionally suppress "Generating catalog" messages during `make -s` builds.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option to mirror Autotools’ `USE_NSEC` was **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux filesystems. The discussion now centers on whether the default should flip to `true` or the knob should be deprecated.

**L10n updates** -- Jiang Xin’s **pull request** for Git 2.55.0 updates `.po` files for 11 languages, including a mass typo-fix pass in the French translation. The changes are **ready for merging**.

**Reftable writer leak fix** -- Jeff King’s **follow-up** resolved a textual conflict in the reftable memory-leak fix, ensuring the `hash_id` initialization remains grouped with other `opts` setup.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit-dropping issue before the next release.
- **`git history squash`**: Harald Nordgren’s **v6 series** is ready for Junio’s final review.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw. The series’ future may depend on Taylor’s availability after a role change.
- **`git history --reword` leak fix**: Junio’s patch addressing a file stream leak in `git history --reword` is **merge-ready**.
- **`--track=fetch` debate**: Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on workflow concerns.