# Here is the digest for **2026/06/27 (Saturday)**:

---

### The day in brief
A Saturday with **50 emails across 15 threads**, lighter than a weekday but still active. The standout was **Johannes Schindelin’s regression fix** for HTTPS proxy URLs, already merged to `master`. Two long-running series—**`git replay --linearize` (v6)** and **`git log -L` range-scoped diff stat/whitespace (v2)**—posted new iterations, while a **security inquiry about OpenSSL CVE-2026-34182** closed with confirmation that Git is unaffected. A **`git ls-remote` hang** thread shifted from a busy-loop to a timeout, leaving the root cause unresolved.

---

### Notable threads

#### **HTTPS proxy regression fixed**
**Subject:** http: fix regression rejecting HTTPS proxy URLs
Johannes Schindelin identified and fixed a **regression in commit `663d7abe07ea`** that incorrectly rejected HTTPS proxy URLs (`https://...`) as unsupported. The bug was a control-flow error in `set_curl_proxy_type()` where the HTTPS branch fell through to a `return -1` intended for unknown schemes. The two-line patch adds an explicit `return 0` after HTTPS setup, mirroring the behavior for HTTP and SOCKS proxies. Junio C Hamano initially suggested refactoring the function to return `void` but retracted the idea after recognizing the need to reject unsupported schemes. The fix is **merged to `master`** ahead of Git 2.55.

---

#### **`git replay --linearize` v6 posted**
**Subject:** [PATCH 0/3] replay: introduce --linearize option
Toon Claes posted **v6** of the `--linearize` option for `git replay`, addressing all prior feedback. The series flattens merge commits into a linear history, with a **regression fix** in this iteration: restoring the `replayed_base` parameter to ensure all replayed commits are linearized into a single topology. The **interface design debate**—whether to mirror `git rebase`'s `--rebase-merges=<mode>` syntax—remains unresolved but is no longer blocking. Junio C Hamano suggested justifying the `--linearize`/`--revert` incompatibility or moving it to a BUGS section, while Phillip Wood clarified the behavioral difference from `git rebase --no-rebase-merges`. The series is **technically complete** and ready for review.

---

#### **`git log -L` range-scoped diff stat/whitespace v2**
**Subject:** [PATCH 0/7] line-log: support range-scoped diff stat, whitespace check, and -G pickaxe
Michael Montalbo (on behalf of Karthik Nayak) posted **v2** of the series extending `git log -L` to support `--stat`, `--numstat`, `--shortstat`, `--check`, and `-G` scoped to tracked line ranges. The update rewords documentation to avoid "range-scoped" jargon, addressing Junio’s feedback. The series is **complete**, with all seven patches implemented, documented, and tested. No new objections have surfaced, but the complexity of the line-range filter may attract further scrutiny.

---

#### **Security inquiry: OpenSSL CVE-2026-34182**
**Subject:** Security inquiry: CVE-2026-34182 (OpenSSL 3.5.6 in Git 2.54.0)
A reported critical CVE in OpenSSL 3.5.6 prompted an inquiry about Git’s exposure. **Todd Zullinger** clarified that Git’s OpenSSL usage does not invoke the CMS functionality affected by the CVE, and **Johannes Schindelin** confirmed that the OpenSSL executable bundled with Git for Windows is not critical. The thread **closed with no action required**: Git itself is unaffected, and the next Git for Windows release (v2.55.0) will include OpenSSL 3.5.7.

---

#### **`git ls-remote` hang: busy-loop to timeout**
**Subject:** `git ls-remote` hangs with 100 % CPU on specific repository
Steffen Nurpmeso reported a **busy-loop in `git ls-remote`** against `https://gitlab.xiph.org/xiph/opus.git`, which later shifted to a low-speed timeout. **Michael Montalbo** suggested `GIT_TRACE_CURL=1` and configuration tweaks (`http.version=HTTP/1.1`, `http.lowSpeedLimit`, `http.lowSpeedTime`) to work around potential network issues. The root cause remains unidentified, but the original symptom has not recurred. The thread highlights a potential edge case in Git’s HTTP transport layer under constrained network conditions.

---

### In brief

**`greplint.pl` test suite conversion** -- Michael Montalbo’s series to convert bare `grep` assertions to `test_grep` surfaced a **pre-existing bug** in `t3420-rebase-autostash.sh`, where `file3` should not exist after `git rebase --quit`. Junio proposed extending `test_grep` to detect missing files, which could address both the bug and future automation risks.

**`git cat-file --batch-command` remote object-info** -- Pablo Sabater addressed **Karthik Nayak’s feedback** on the GSoC series, clarifying error-handling philosophy (silent continuation vs. explicit failure) and transport-layer integration. The series is **nearing final readiness**, with only minor documentation nits remaining.

**`excludes_file` libification** -- Tian Yuchen posted **v3** of the patch moving the global `excludes_file` variable into `struct repo_config_values`. The update consolidates prior work, adds a destructor, and aligns with the project’s libification goals. Junio raised a **guardrail debate** about silent early returns vs. `BUG()` assertions, but the series is otherwise ready for `next`.

**`git repack --geometric --cruft` RFC** -- Taylor Blau’s RFC to combine `--geometric` and `--cruft` modes hit a **correctness snag**: the two-phase traversal in `--stdin-packs=follow-reachable` may retain unreachable tags. Taylor proposed reversing the traversal order but has not yet implemented the fix. The series remains in RFC stage pending further iteration.

**Usability improvements for branch/push mistakes** -- Harald Nordgren’s **v3** series to improve error messages for `git branch --set-upstream-to` and `git push` misformatting is **queued in `seen`**. The patches add config-based advice (`advice.setUpstreamFailure`, `advice.pushRepoLooksLikeRef`) and are ready for `next`.

**Gitk/Git-GUI quiet build alignment** -- Johannes Sixt confirmed that the patches to silence translation-catalog build output during `make -s` are in his testing branches and will be proposed for upstream Git inclusion in the coming weeks.

---

### On the radar
- **`git replay --linearize`**: Junio’s documentation nit about `--linearize`/`--revert` incompatibility may resurface during review.
- **`git repack --geometric --cruft`**: The correctness issue in `--stdin-packs=follow-reachable` needs a concrete fix before the series can progress.
- **`git ls-remote` hang**: The root cause remains unidentified; no similar reports have surfaced, but the issue may reappear under specific network conditions.