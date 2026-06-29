# The Git Mailing List Digest for 2026/06/28

**The day in brief.** A Sunday with moderate traffic (44 emails across 16 threads) saw the resolution of several long-running efforts: the `git replay --linearize` regression was identified, the `paint_down_to_common()` optimization series reached final approval, and a usability series for `git branch`/`git push` mistakes was queued for merging. The most urgent item is a **critical regression in `git replay --linearize`** causing silent commit dropping, which now needs a follow-up patch.

---

## Notable threads

### Critical regression in `git replay --linearize` causes silent commit dropping
Johannes Schindelin identified a **regression in the recently merged `git replay --linearize` feature** that causes commits to be silently dropped when replaying a single branch containing merge commits. The issue stems from a v5 fix for `--onto` edge cases that inadvertently removed the `replayed_base` logic, which was designed to preserve linearized history. Schindelin provided a concrete test case (`master~2..master`) showing that only the tip commit is replayed, while intermediate commits (including merges) are lost. This is a **high-priority issue** requiring a follow-up patch to restore the correct behavior before the next release. The series was merged just two weeks ago, so the regression is fresh and unshipped.

---

### `paint_down_to_common()` optimization series reaches final approval
Derrick Stolee **approved the entire 8-patch series** optimizing `paint_down_to_common()` for one-sided histories, marking the end of a months-long effort. The series delivers **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts or shallow histories) by terminating early when one side of the walk is exhausted. Key changes in v4 include a **regression fix** (restoring the `min_generation` guard to the BUG assertion), **renamed counters for clarity**, **consolidated termination conditions**, and **expanded test coverage** for edge cases. The series is now **ready for merging** and represents a significant performance win for large repositories.

---

### Usability series for `git branch`/`git push` mistakes queued for merging
Junio C Hamano queued Harald Nordgren’s **2-patch usability series** that improves error messages for common command-line mistakes:
1. `git branch --set-upstream-to=origin main` (missing slash) now suggests `origin/main` when the remote-tracking ref exists.
2. `git push origin/main` (slash instead of space) now guides users toward the correct syntax.
The series introduces two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and is **ready for the next integration cycle**. The patches address real-world usability pain points and align with Git’s broader effort to make error messages more actionable.

---

### `git history squash` series reaches v6, addressing all prior feedback
Harald Nordgren posted **v6 of the `git history squash` series**, a new subcommand that folds a commit range into its oldest commit while preserving descendant history. The series now supports **multiple revision arguments** (e.g., `@~3.. ^topic`), stricter input validation, and the `--reedit-message` flag to combine all folded-in commit messages in the editor. The implementation uses a single three-way merge of the entire range against the oldest commit’s parent, followed by replaying descendants. The series is **feature-complete and ready for final review**, with no further design changes expected.

---

### CI hangs in `t5551`/`t5559` resolved with Apache timeout increase
Jeff King (Peff) posted a **2-patch series** addressing macOS CI hangs in `t5551` and `t5559` caused by Apache bug 70131. The core fix increases Apache’s `Timeout` directive from 300 to 600 seconds, directly addressing the root cause of HTTP/2 stalls and `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. A complementary patch isolates the expensive test case into its own repository, reducing runtime for subsequent tests. The series is **low-risk and likely to be merged quickly**, as the technical direction has already been agreed upon in the thread.

---

### `USE_NSEC` re-evaluated: modern Linux preserves nanosecond timestamps
Jeff King’s testing revealed that **modern Linux filesystems (ext4, ext2, vfat, CIFS) preserve nanosecond timestamps correctly**, obsoleting the original rationale for the `USE_NSEC` knob. However, he also identified **interoperability risks** when mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit), which could degrade performance. The discussion has shifted from "should Meson expose this option?" to "should the knob be deprecated or flipped to `true` by default?" No patch changes have been proposed yet, but the findings challenge the assumption that `USE_NSEC` is inherently problematic.

---

## In brief

**`git replay --linearize` regression follow-up needed** -- Johannes Schindelin identified a critical regression in the recently merged `git replay --linearize` feature, causing silent commit dropping when replaying single branches with merge commits. A follow-up patch is urgently required to restore the `replayed_base` logic before the next release.

**`paint_down_to_common()` optimization approved** -- Derrick Stolee approved the 8-patch series optimizing `paint_down_to_common()` for one-sided histories, delivering 100-1000x speedups for asymmetric merge-base queries. The series is ready for merging.

**Usability series for `git branch`/`git push` queued** -- Junio C Hamano queued Harald Nordgren’s 2-patch series improving error messages for common command-line mistakes (e.g., `git branch --set-upstream-to=origin main`). The series is ready for the next integration cycle.

**`git history squash` v6 posted** -- Harald Nordgren posted v6 of the `git history squash` series, addressing all prior feedback. The series is feature-complete and ready for final review.

**CI hangs in `t5551`/`t5559` resolved** -- Jeff King posted a 2-patch series increasing Apache’s `Timeout` directive to address macOS CI hangs caused by Apache bug 70131. The series is likely to be merged quickly.

**`USE_NSEC` re-evaluated** -- Jeff King’s testing showed that modern Linux preserves nanosecond timestamps correctly, challenging the original rationale for the `USE_NSEC` knob. The discussion now centers on whether the knob should be deprecated or flipped to `true` by default.

**Test suite bugfix queued** -- A fragile test in `t/t3420-rebase-autostash.sh` was fixed and queued by Junio C Hamano. The patch replaces `! grep dirty file3` with `test_path_is_missing file3` to correctly assert file absence.

**`excludes_file` migration ready for `next`** -- The migration of the global `excludes_file` variable into `struct repo_config_values` is ready for `next` after resolving the guardrail design debate. The patch centralizes initialization checks in `repo_config_values()`.

**HTTPS proxy regression fixed** -- Johannes Schindelin’s fix for a regression rejecting HTTPS proxy URLs was merged. The patch adds a missing `return 0` in `set_curl_proxy_type()`.

**Reftable memory leak fixed** -- Jeff King fixed a memory leak in `reftable_writer_new` where the `reftable_writer` struct was allocated before input validation. The patch was merged with a trivial conflict resolution.

**L10n updates for Git 2.55.0** -- Jiang Xin posted a pull request updating `.po` files for 11 languages, plus two infrastructure tweaks. The updates are routine and ready for merging.

---

## On the radar

**`greplint.pl` tooling risks** -- Junio C Hamano and Michael Montalbo are debugging why the `greplint.pl` tool preserves latent test bugs, exposing a tension between automation and diagnostic rigor. The discussion may lead to a redesign of the linter’s exemption mechanism.

**Worktree diff performance** -- Jeff King and Junio C Hamano are discussing whether to make `USE_NSEC` unconditional on Linux to address worktree diff slowdowns. The edge case (kernel inode cache eviction) is considered unlikely but untested.