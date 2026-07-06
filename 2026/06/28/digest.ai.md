# The Git Project Daily Digest – 2026/06/28 (Sunday)

## The day in brief
A **moderate-volume Sunday** (45 emails, 17 threads) saw **two major series land**, a **post-merge regression surface**, and **long-running debates advance**. The standout developments: **Harald Nordgren’s `git history squash` v6** (4 patches) reached feature-complete status, **Tian Yuchen’s `paint_down_to_common()` optimization v4** (8 patches) was fully approved after fixing a regression, and **Johannes Schindelin reported a critical regression in `git replay --linearize`** that silently drops commits. The day also brought **final approval for usability improvements** in `git branch` and `git push`, and **a consensus fix for macOS CI hangs** in `t5551`/`t5559`.

---

## Notable threads

### `git history squash` v6 lands – feature-complete and ready for review
Harald Nordgren’s **four-patch series** implementing `git history squash` reached **feature-complete status in v6**, addressing all prior feedback and adding support for **multiple revision arguments** (e.g., `@~3.. ^topic`). The command folds a commit range into its oldest commit while preserving descendant history, providing a faster alternative to `git rebase --no-rebase-merges` for linearizing histories.

The series now **rejects single-commit ranges** (e.g., `@^!`) and **non-ancestor ranges**, and includes **stricter input validation** and **expanded test coverage** (550+ lines). The `--reedit-message` flag gathers all folded-in commit messages (oldest first) into an editor template, mirroring `git rebase -i` behavior. The ref-handling logic remains unchanged: the command rejects operations where any ref points to a commit inside the squashed range, advising users to use `--update-refs=head` to retarget only the current branch.

The implementation is **well-tested and ready for Junio’s final review**, with no open design questions. The series touches `builtin/history.c`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, and a new test script (`t/t3455-history-squash.sh`). The only remaining action is Junio’s assessment; no further changes are expected.

---

### `paint_down_to_common()` optimization v4 fully approved after regression fix
Tian Yuchen’s **eight-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged** after fixing a **critical regression** in v3. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). The optimization delivers **100-1000x speedups** for asymmetric queries, with benchmarks showing step-count reductions from 72,264 to 44,599 for `merge-base --all v2.0.0 v2.55.0-rc1`.

The v4 iteration **fixed the regression** (an unconditional BUG assertion that broke correctness when `min_generation` was not set) and **consolidated termination conditions** in `paint_queue_get()`. The series also includes **trace2 instrumentation** (patch 4/8), **edge-case test coverage** (patches 2/8 and 3/8), and a **new technical document** (`paint-down-to-common.adoc`). Derrick Stolee’s **unqualified approval** for the entire series removes the last open technical question; the patches are now **ready for merging**.

The changes are **internal to the commit-reach subsystem** and have no on-disk format or user-visible behavior changes. The test suite robustly verifies correctness across all commit-graph modes (none, full, half, no-gdat) and edge cases (self, duplicate-twos, pending-stale, infinity-both-sides).

---

### `git replay --linearize` regression: silent commit dropping
Johannes Schindelin reported a **post-merge regression** in `git replay --linearize` (merged in v5) that **silently drops commits** when replaying a single branch containing merge commits. The issue occurs in a test case where replaying `master~2..master` with `--linearize --onto master~2` replays the tip commit directly onto the `--onto` target, omitting the first replayed commit ("Git 2.55-rc2"). Schindelin traced the problem to the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact commit-dropping behavior.

The regression is **urgent** and must be fixed before the next release. Toon Claes is expected to send a follow-up patch to restore the `replayed_base` logic or redesign how multi-branch histories are handled. The issue highlights the complexity of linearizing histories with merge commits, particularly when replaying three branches where one is a merge of the other two. Schindelin’s review is **technically precise** and includes a concrete reproduction scenario, making it actionable for a fix.

---

### Usability improvements for `git branch` and `git push` graduate to `next`
Harald Nordgren’s **two-patch usability series** improving error messages for common command-line mistakes was **approved and queued for `next`**. The patches address:
1. **`git branch --set-upstream-to=origin main`** (missing slash): The command now suggests the correct `<remote>/<branch>` form when the remote-tracking ref (`origin/main`) exists.
2. **`git push origin/main`** (slash instead of space): The command now validates the remote portion and excludes existing paths/repos.

The series introduces two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and includes **thorough test coverage** in `t/t3200-branch.sh` and `t/t5529-push-errors.sh`. Junio C Hamano’s final review noted a minor design quibble (redundant advice checks) but declared it too small to warrant a rewrite. The patches are **ready for merging** and will improve the user experience for common workflow mistakes.

---

### macOS CI hangs in `t5551`/`t5559`: consensus fix implemented
Jeff King (Peff) sent a **three-patch series** implementing the consensus fix for macOS CI hangs in `t5551` and `t5559`. The series addresses Apache bug 70131, which causes HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. The patches:
1. **Increase Apache’s `Timeout` directive** from 300 to 600 seconds in `t/lib-httpd/apache.conf`.
2. **Isolate the expensive "many-tags" test case** into a dedicated repository (`many-tags.git`).
3. **Pack refs after creating many tags** to reduce `ls-refs` advertisement time from ~1.2 seconds to 24 ms.

The first patch is the **core fix**, directly addressing the root cause (Apache’s 300-second timeout being hit during slow `upload-pack` responses). The second and third patches are **optimizations** that reduce timeout risk, though Peff expressed mild skepticism about the third patch’s necessity. Junio C Hamano noted a minor stylistic nit in the second patch (non-bare repository initialization) but did not object to the series. The patches are **ready for merging** and will eliminate the persistent CI flake.

---

## In brief

**Reftable memory leak fix** -- Jeff King fixed a memory leak in `reftable/writer.c` where the `reftable_writer` struct was allocated before input validation. The fix moves the allocation after the block-size check, plugging the leak and improving code hygiene. The issue was introduced in February 2025 and recently flagged by Coverity.

**`excludes_file` libification** -- Tian Yuchen and Junio C Hamano resolved the last design question for the `excludes_file` migration series. The getter `repo_excludes_file()` will now **rely on `repo_config_values()` to enforce initialization**, removing a redundant `repo->initialized` check. The series is **ready for `next`** and aligns with the broader `the_repository` removal effort.

**HTTPS proxy regression fix** -- Johannes Schindelin’s patch fixing a regression in HTTPS proxy URL validation was **merged into `master`**. The fix adds a missing `return 0` in `set_curl_proxy_type()` to correctly accept HTTPS proxy URLs, which were incorrectly rejected after commit `663d7abe07ea`.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option was **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. Peff’s findings show that nanosecond timestamps are preserved correctly on ext4, ext2, vfat, and CIFS, contradicting earlier concerns about coarse-grained filesystems. The discussion now centers on whether the default should flip to `true`.

**French translation update** -- Jiang Xin’s **l10n pull request** updates the `.po` files for 11 languages and includes a mass typo-fix pass for French. The request is **ready for merging** and covers new and changed strings in Git 2.55.0.

**`git ls-remote` hang resolved** -- Steffen Nurpmeso reported that the original `git ls-remote` hang (100 % CPU) has not recurred, though the root cause remains unidentified. The thread is now closed from the user’s perspective.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue before the next release.
- **`git history squash` v6**: Harald Nordgren’s series is ready for Junio’s final review.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.