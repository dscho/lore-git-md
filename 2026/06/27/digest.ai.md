# The Git Mailing List Daily Digest for 2026/06/27

**The day in brief.**
Saturday’s traffic (50 emails in 15 threads) was a mix of quiet weekend maintenance and lingering technical debates. The standout was a **security inquiry** about CVE-2026-34182 in OpenSSL 3.5.6, quickly resolved as non-applicable to Git, and a **substantive review** of Taylor Blau’s RFC series combining `--geometric` and `--cruft` repacks, where Junio C Hamano identified a correctness flaw in the reachability filtering logic. A long-dormant test fix resurfaced and looks ready to merge.

---

## Notable threads

### Security inquiry: CVE-2026-34182 (OpenSSL 3.5.6 in Git 2.54.0)
Tim Person asked whether Git 2.54.0’s bundled OpenSSL 3.5.6 exposed users to CVE-2026-34182, a critical CMS decryption/encryption vulnerability. Todd Zullinger clarified that Git’s OpenSSL usage (TLS for fetch/push) does not invoke CMS functionality, and the only potential exposure—via `gpgsm` for x509 signatures—is not impacted by this CVE. The Windows installer question was redirected to the separate Git for Windows project. The thread closed with no action required from the Git project.

### `git repack`: combining `--geometric` and `--cruft` (RFC v1)
Taylor Blau’s 10-patch RFC series to allow `git repack` to combine `--geometric` and `--cruft` modes hit a snag. Junio C Hamano’s review of patch 8/10 identified a **correctness issue** in the two-phase traversal logic of `--stdin-packs=follow-reachable`: the current implementation marks *all* tag objects found in included packs as reachable, regardless of whether they are actually reachable from refs. This risks including unreachable objects in the output pack, undermining the series’ goal of cleanly separating reachable and unreachable objects. Taylor acknowledged the flaw and proposed reversing the traversal order to prioritize reachability from refs, but no concrete implementation has been submitted yet. The series remains in RFC stage, with further iteration expected.

### Test fix: fragile `git rebase --quit` test
SZEDER Gábor resurfaced a five-year-old patch fixing a fragile test in `t/t3420-rebase-autostash.sh`. The test incorrectly used `! grep dirty file3` to verify the absence of dirty content after `git rebase --quit`, but the file *should not exist at all* in that scenario. The patch replaces the `grep` check with `test_path_is_missing file3`, aligning with Git’s test suite conventions. Phillip Wood endorsed the fix, and Todd Zullinger flagged a commit message typo ("Thighten" → "Tighten"). The patch is now ready for merging, with no unresolved technical concerns.

### `git replay --linearize`: post-merge behavioral edge case
Phillip Wood confirmed Junio C Hamano’s post-merge observation about `git replay --linearize`: the command does not flatten history like `git rebase --no-rebase-merges`. Instead, it grafts only the first-parent branch onto the new history, leaving the other branch dangling. Phillip’s reply clarified the mechanism—replay cherry-picks each commit onto its rewritten parent, whereas rebase replays all commits in a linear sequence—without judging whether the behavior is correct. The divergence is now recognized as a **design limitation** rather than a bug, and a follow-up patch to document the behavior or add user control (e.g., `--first-parent=<branch>`) is likely.

### `git log -L`: range-scoped diff stat, whitespace check, and `-G` pickaxe (v2)
Karthik Nayak submitted v2 of a seven-patch series extending `git log -L` to support range-scoped diff stat formats (`--stat`, `--numstat`, `--shortstat`), whitespace checking (`--check`), and the `-G` pickaxe. The series reuses the line-range filter infrastructure to scope stat counts, whitespace errors, and `-G` matches to the tracked range, while deliberately excluding `--dirstat` and `-S` due to architectural constraints. Junio’s feedback on patch 5 (documentation phrasing) has been incorporated, and the series is now complete with all patches implemented, documented, and tested. The implementation is clean and well-motivated, with thorough test coverage in `t/t4211-line-log.sh`.

---

## In brief

**`excludes_file` libification (v4)** -- Tian Yuchen’s patch moves the global `excludes_file` variable into `struct repo_config_values` as part of the ongoing libification effort. The patch introduces a getter (`repo_excludes_file()`) and a destructor (`repo_config_values_clear()`) to manage heap-allocated memory, and uses `repo->initialized` checks for consistency with prior work. Junio C Hamano proposed replacing the silent early return with a `BUG()` assertion for stricter fail-fast behavior, but the patch is otherwise ready for `next`.

**Usability improvements for `git branch` and `git push` (v3)** -- Harald Nordgren’s two-patch series improves error messages for common command-line slips: `git branch --set-upstream-to=origin main` (missing slash) and `git push origin/main` (slash instead of space). The series is complete, with all feedback addressed, and is queued in `seen` for graduation to `next`.

**`gitk` and `git-gui` quiet build output** -- Johannes Sixt confirmed that the changes to silence translation-catalog build output during `make -s` for gitk and git-gui are in his testing branches and will be proposed for upstream Git inclusion in the coming weeks. The changes align with core Git’s quiet build conventions.

**`git ls-remote` hang resolved** -- Steffen Nurpmeso reported a transient infinite loop in `git ls-remote` against `https://gitlab.xiph.org/xiph/opus.git`, but the issue has since disappeared. The root cause remains unidentified, though external factors (e.g., Anubis proxy rate-limiting) were suspected.

**HTTP proxy regression fix** -- Johannes Schindelin fixed a regression in `663d7abe07ea` where Git incorrectly rejected HTTPS proxy URLs (`https://...`) as unsupported. The fix adds an explicit `return 0` in `set_curl_proxy_type()` to prevent fall-through to the error case.

---

## On the radar

**`greplint.pl` test suite conversion** -- Michael Montalbo’s series to convert bare `grep` assertions to `test_grep` in Git’s test suite faces methodological scrutiny. Junio C Hamano proposed extending `test_grep` to detect missing-file cases (e.g., the `t3420-rebase-autostash.sh` bug), which could address maintainers’ concerns about automation risks while preserving the tool’s value. The series’ adoption path hinges on whether this enhancement is feasible.

**Source tree reorganization (RFC v2)** -- Patrick Steinhardt’s RFC to move `libgit.a` components into a new `lib/` directory continues to draw debate. SZEDER Gábor raised a **user-facing concern**: the reorganization disrupts `git log --follow` and exacerbates the "rename barrier" problem, making it harder to trace file history across the move. The discussion has shifted from implementation details to whether the benefits (e.g., structural clarity) justify the costs. Junio remains skeptical about the practical benefits, particularly during the prerelease freeze period.