## The day in brief

Saturday, June 27, 2026 was a **moderately busy** day on the Git mailing list, with **50 emails across 15 threads**. The standout developments were **two long-running series reaching technical completion**—Michael Montalbo’s `greplint.pl` test-suite linter and Karthik Nayak’s `git log -L` range-scoped diff features—while **a critical correctness flaw** surfaced in Taylor Blau’s `--geometric`/`--cruft` repack series. A **security inquiry** about OpenSSL CVE-2026-34182 was swiftly resolved, and **two user-reported hangs** (one resolved, one still open) kept the support queue active.

---

## Notable threads

### `greplint.pl` test-suite linter reaches final approval
Michael Montalbo’s six-patch series converting bare `grep` assertions to `test_grep` in the test suite is now **technically complete and approved for merging**. Junio C Hamano signed off on v4 after all prior feedback—including concerns about `# lint-ok:` exemptions masking pre-existing bugs—was addressed. The series found **10+ latent test bugs** during conversion, demonstrating its value beyond mere style cleanup. A new discussion emerged about extending `test_grep` to catch missing-file cases (e.g., Gábor’s 2021 `t3420` fix), but this is now a follow-up topic. The series is **ready for `next`** and represents a significant step toward test-suite robustness.

### `git log -L` gains range-scoped diff stat, whitespace checking, and `-G` pickaxe
Karthik Nayak’s seven-patch v2 series extending `git log -L` to support `--stat`, `--check`, and `-G` is now **fully implemented and documented**. The patches reuse the line-range filter infrastructure to scope stat counts, whitespace errors, and pickaxe matches to the tracked range, while deliberately excluding `--dirstat` and `-S` due to architectural constraints. Junio’s feedback on patch 5 (documentation phrasing) has been incorporated, and the series is **ready for review**. The implementation is clean, well-tested, and aligns with the recent `mm/line-log-cleanup` topic. No substantive objections remain, though the exclusion of `-S` may spark discussion about future extensibility.

### `--geometric`/`--cruft` repack series hits critical correctness flaw
Taylor Blau’s RFC series combining `--geometric` and `--cruft` repack modes has **stalled on a non-trivial correctness issue** in the `--stdin-packs=follow-reachable` logic. Junio C Hamano identified that the two-phase traversal (marking objects in included packs, then walking from reference tips) may retain unreachable tags and objects, undermining the series’ core goal of separating reachable and unreachable objects. Taylor acknowledged the flaw and proposed reversing the traversal order, but no concrete implementation has been submitted yet. The series remains in RFC stage, with **further iteration expected** before it can progress. The flaw is a reminder of the complexity of reachability filtering in large repositories.

### OpenSSL CVE-2026-34182 inquiry resolved
A security inquiry about CVE-2026-34182 (OpenSSL 3.5.6) in Git 2.54.0 was **swiftly closed** after Todd Zullinger and Johannes Schindelin clarified that Git’s OpenSSL usage does not invoke the CMS functionality affected by the CVE. The OpenSSL executable bundled with Git for Windows is not critical and will be updated in the next scheduled release (v2.55.0, June 29–30). No Git release or advisory is planned, and the thread is now closed.

---

## In brief

**`git replay --linearize` regression fix** -- Toon Claes’s v6 series fixing a regression in `--linearize` behavior when replaying a single branch containing merge commits is now **technically complete**. The `replayed_base` parameter has been restored to ensure all replayed commits are flattened into a single topology, and the behavioral difference from `git rebase --no-rebase-merges` is documented and tested. The series is ready for review, with no unresolved technical objections.

**`git branch`/`git push` usability improvements** -- Harald Nordgren’s v3 series improving error messages for common command-line slips (e.g., `git branch --set-upstream-to=origin main` or `git push origin/main`) is **queued in `seen`** and slated for `next`. The series introduces two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and helper functions to detect and suggest corrections. All feedback from v1 and v2 has been addressed, and the implementation is clean and well-tested.

**`gitk`/`git-gui` quiet build integration** -- Harald Nordgren’s patch silencing translation-catalog build output during `make -s` for gitk and git-gui is **merged into Johannes Sixt’s testing branches** and will be proposed for upstream Git inclusion in the coming weeks. The changes align with core Git’s quiet build conventions and are procedurally complete.

**`excludes_file` libification** -- Tian Yuchen’s v4 patch moving the global `excludes_file` variable into `struct repo_config_values` is **ready for review**. The patch consolidates earlier work, introduces a getter and destructor for memory management, and aligns with the broader libification effort. All prior feedback has been addressed, and the patch is mentored by experienced contributors.

**HTTP proxy regression fix** -- Johannes Schindelin’s two-line fix for a regression rejecting HTTPS proxy URLs is **merged into `master`**. The patch corrects a control-flow error in `set_curl_proxy_type()` and preserves the original hardening intent. No tests were added, as the change addresses a specific logic error rather than introducing new behavior.

**`git ls-remote` hang resolved** -- A user-reported infinite loop in `git ls-remote` against `https://gitlab.xiph.org/xiph/opus.git` has **disappeared**, with the command now completing normally. The original busy-loop symptom was replaced by a low-speed timeout under constrained network conditions, but no root cause inside Git was identified. The thread is closed from a user-support standpoint.

---

## On the radar

**`git ls-remote` hang under constrained network conditions** -- A second user-reported hang in `git ls-remote` (this time against an unspecified repository) remains **unresolved**. The command now fails with a low-speed timeout unless `http.lowSpeedLimit` and `http.lowSpeedTime` are adjusted, but the root cause inside Git is still unknown. The issue is tied to constrained network conditions (possibly involving a proxy) and may require further debugging with `GIT_TRACE_CURL=1`. No similar reports from other users have been recorded, so this remains a low-priority but open investigation.