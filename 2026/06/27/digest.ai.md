## The day in brief

Saturday, June 27, 2026 was a moderately busy day on the Git mailing list, with 50 emails across 15 active threads. The most consequential developments were **substantive review feedback** on two long-running series—**`git cat-file --batch-command`’s remote object-info feature** and **`git repack`’s `--geometric`/`--cruft` combination**—that threaten to delay their merging. A **new bug report** about `git ls-remote` hanging at 100% CPU also surfaced, though it appears transient. Meanwhile, **two usability improvements** (`git branch --set-upstream-to` and `git push` error messages) and a **libification refactoring** (`excludes_file` migration) reached final readiness.

---

## Notable threads

### `git cat-file --batch-command` remote object-info series hits architectural snag
Pablo Sabater’s 13-patch GSoC series, which adds `remote-object-info` support to `git cat-file --batch-command`, received **substantive review feedback** from Karthik Nayak on the client-side implementation (patch 10/13). The critique centers on **error-handling philosophy**—whether the client should silently skip objects lacking size data (Pablo’s approach) or fail explicitly (Karthik’s preference)—and **transport-layer integration**, where the patch piggybacks on `fetch_refs_via_pack()` rather than adding a dedicated vtable entry. Pablo’s responses defend the silent-failure design as consistent with local `git cat-file` behavior but concede the transport-layer awkwardness, planning to move the logic to a dedicated vtable entry in the next revision. The **error-handling debate** may require broader discussion, as it touches on user expectations for partial failures. The series remains **technically complete** but is now blocked on these architectural adjustments.

---

### `git repack --geometric --cruft` series stumbles on reachability flaw
Taylor Blau’s 10-patch RFC series, which allows `git repack` to combine `--geometric` and `--cruft` modes, hit a **critical correctness issue** in the `--stdin-packs=follow-reachable` implementation (patch 8/10). Junio C Hamano’s review identified a flaw in the two-phase traversal logic: the current code marks *all* tag objects found in included packs (or loose objects) as starting points for the second-phase traversal, regardless of whether they are reachable from refs. This risks including unreachable objects in the output pack, undermining the series’ core goal of separating reachable and unreachable objects. Taylor acknowledged the issue and proposed reversing the traversal order (prioritizing reachability from refs before marking objects in included packs), but no concrete fix has been implemented yet. The series is **not ready for merging**, and further iteration is expected to resolve this reachability flaw.

---

### `git replay --linearize`’s merge divergence behavior clarified
The recently merged `git replay --linearize` series (Toon Claes) saw **post-merge discussion** about its handling of merge commits. Phillip Wood clarified that `git replay --linearize` drops the merge and grafts only the first-parent branch onto the new history, leaving the other branch dangling, whereas `git rebase --no-rebase-merges` replays both branches. Junio proposed adding a test to document this behavior, which Toon is likely to incorporate. The discussion does not dispute the implementation’s correctness but highlights a **user-expectation mismatch** that may require follow-up documentation or behavioral adjustments.

---

### `excludes_file` libification refactoring reaches final readiness
Tian Yuchen’s refactoring series, which moves the global `excludes_file` variable into `struct repo_config_values` as part of the ongoing libification effort, reached **final readiness** in its fourth iteration. The patch consolidates two earlier commits, replaces a controversial guard with a stricter `repo->initialized` check, and adds proper memory management via an extended `repo_config_values_clear()` helper. Junio’s only remaining concern—a defensive programming style in the new getter—is unlikely to block merging, as the patch aligns with established patterns and has mentorship from Christian Couder. The series is **poised for `next`** and represents a clean, incremental step in the `the_repository` removal effort.

---

### Usability improvements for `git branch` and `git push` error messages land
Harald Nordgren’s two-patch series improving error messages for common command-line slips (`git branch --set-upstream-to=origin main` and `git push origin/main`) reached **final readiness** in v3. The patches add config-based advice (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) that triggers only when the remote-tracking ref exists (branch case) or the remote is configured (push case), reconstructing the user’s likely intent. Junio queued the series in `seen`, and it is slated for `next`. The implementation is **uncontroversial**, with only a non-blocking edge case (where the local branch exists but the remote-tracking ref also exists) noted for future follow-up.

---

## In brief

**`git log -L` range-scoped diff stat and whitespace checking** -- Michael Montalbo sent v2 of Karthik Nayak’s seven-patch series extending `git log -L` to support `--stat`, `--numstat`, `--shortstat`, `--check`, and `-G`. The only change from v1 is reworded documentation to avoid "range-scoped" jargon, addressing Junio’s feedback. The series is **ready for review**, with no outstanding technical objections.

**Test suite fix for `t3420-rebase-autostash.sh`** -- SZEDER Gábor resurfaced a five-year-old patch fixing a fragile test in `t3420-rebase-autostash.sh`, replacing `! grep dirty file3` with `test_path_is_missing file3` to properly assert the file’s absence after `git rebase --quit`. Phillip Wood endorsed the fix, and Todd Zullinger noted a typo in the commit message. The patch is **ready for merging**, with no unresolved concerns.

**`git ls-remote` infinite loop bug report** -- Steffen Nurpmeso reported a **100% CPU hang** in `git ls-remote` when querying `https://gitlab.xiph.org/xiph/opus.git`, but the issue disappeared before further investigation. Michael Montalbo suggested network-related workarounds, but the root cause remains unidentified. The report is **closed as transient**, though the lack of a stack trace or bisect results leaves it unresolved.

**HTTP transport regression fix** -- Johannes Schindelin fixed a regression in the HTTP transport layer where Git incorrectly rejected HTTPS proxy URLs. The fix adds an explicit `return 0` in `set_curl_proxy_type()` to prevent falling through to an error return. The patch is **small, self-contained, and ready for merging**.

**Security inquiry about CVE-2026-34182** -- Tim Person inquired about Git’s plans to update its bundled OpenSSL 3.5.6 executable for CVE-2026-34182. Todd Zullinger clarified that the Git project does not distribute binaries (redirecting to Git for Windows) and noted that the CVE’s CMS-specific nature likely does not affect Git’s usage of OpenSSL. The thread **closed with no action required**.

---

## On the radar

**`greplint.pl` series faces methodological objections** -- Michael Montalbo’s `greplint.pl` tool for converting bare `grep` assertions to `test_grep` in Git’s test suite remains stalled on maintainer objections. Junio and SZEDER Gábor raised concerns about automated conversion masking latent bugs (e.g., a pre-existing bug in `t3420-rebase-autostash.sh` preserved by the linter). Junio proposed extending `test_grep` to detect missing files, but the broader question of whether the tool should convert or merely diagnose remains unresolved. The series’ demonstrated value (10+ bugfixes) clashes with maintainers’ preference for diagnostic rigor.