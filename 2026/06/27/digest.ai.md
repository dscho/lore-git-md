## The day in brief

Saturday, June 27, 2026 was a **moderate-traffic day** (50 emails, 15 threads) with a **mix of bug fixes, refactoring, and user support**. The standout developments were **Junio’s approval of the `greplint.pl` test-suite linter** (now merged to `next`), **Taylor Blau’s RFC series combining `--geometric` and `--cruft` repack modes** (with a critical correctness flaw identified), and **a security inquiry about OpenSSL 3.5.6** (resolved as non-applicable to Git). The day also saw **final iterations of long-running refactoring efforts** (e.g., `excludes_file` libification) and **user reports of transient hangs** in `git ls-remote`.

---

## Notable threads

### `greplint.pl` test-suite linter approved for merging
Michael Montalbo’s six-patch series introducing `greplint.pl`—a linter that converts bare `grep` assertions to `test_grep` for improved debuggability—**reached final approval and was merged to `next`**. The series, which found and fixed 10+ pre-existing test bugs, had faced maintainer skepticism about `# lint-ok:` exemptions preserving incorrect logic. Junio’s final review accepted the shift to explicit guards (e.g., `test_have_prereq REFFILES`) and the documentation updates, resolving all objections. The linter’s value was underscored by a **latent bug in `t3420-rebase-autostash.sh`** (where `file3` should not exist after `--quit`) resurfaced by SZEDER Gábor, prompting Junio to propose extending `test_grep` to catch such cases. The series is now **technically complete**, with 680 lines of new tests and verified memory safety.

**Key files**: `greplint.pl`, `t/test-lib-functions.sh`, `t/README`.

---

### `--geometric` + `--cruft` repack RFC reveals critical flaw
Taylor Blau’s **10-patch RFC series** enabling `git repack` to combine `--geometric` and `--cruft` modes hit a **major snag** when Junio identified a correctness issue in the two-phase traversal logic of `--stdin-packs=follow-reachable`. The flaw risks retaining unreachable tags and objects, undermining the series’ core goal of separating reachable and unreachable objects. Taylor acknowledged the problem and proposed reversing the traversal order (prioritizing reachability from refs before marking objects in included packs), but no concrete implementation has been submitted yet. The series also introduced `--refs-snapshot=<path>` to synchronize reachability between `pack-objects` and the MIDX bitmap writer, a fix for a pre-existing consistency risk. While patches 1–7 and 9 are low-risk refactorings, **patches 8 and 10 may require significant rework** before merging.

**Key files**: `builtin/repack.c`, `builtin/pack-objects.c`, `repack-cruft.c`.

---

### `excludes_file` libification completes three-phase migration
Tian Yuchen’s patch moving the global `excludes_file` variable into `struct repo_config_values` **fulfilled Junio’s hard requirement for merging** by completing the three-phase guardrail migration (silent return → `BUG()` → no check). The final patch (v6) removed the defensive check entirely, as all unsafe callers had been addressed. Junio’s review of v4 raised a **philosophical question** about silent early returns vs. `BUG()` assertions, but the series was **merged to `next`** after Tian clarified the guard’s redundancy. The patch is a **foundational step for libification**, introducing a getter (`repo_excludes_file()`) and destructor (`repo_config_values_clear()`) for heap-allocated memory.

**Key files**: `dir.c`, `environment.c`, `repository.c`.

---

### Security inquiry: OpenSSL 3.5.6 CVE-2026-34182
A **critical CVE in OpenSSL 3.5.6** (bundled with Git 2.54.0) prompted an urgent inquiry about exposure and release timelines. The thread **closed quickly** after Todd Zullinger and Johannes Schindelin clarified that Git’s OpenSSL usage does not invoke the affected CMS functionality, and the OpenSSL executable bundled with Git for Windows is non-critical. The next Git for Windows release (v2.55.0, June 29–30) will include OpenSSL 3.5.7, but **no Git release or advisory is planned**. The inquiry highlighted the **separation between Git and Git for Windows** and the importance of clarifying bundled dependencies.

---

### `git replay --linearize` series nears completion
Toon Claes’s **`git replay --linearize`** series (v6) addressed all prior feedback, including a **regression fix** for single-branch replay with merge commits. The series now documents the behavioral difference from `git rebase --no-rebase-merges` (replay drops merges and keeps one branch; rebase rewrites both) and justifies the `--linearize` flag’s divergence from rebase’s `--rebase-merges=<mode>` syntax. Junio’s remaining nit—a documentation note about `--linearize`/`--revert` incompatibility—was left unresolved but is non-blocking. The series is **ready for review**, with no outstanding technical objections.

**Key files**: `replay.c`, `Documentation/git-replay.adoc`.

---

## In brief

**`git ls-remote` hang investigation** -- Steffen Nurpmeso reported a transient infinite loop in `git ls-remote` against `https://gitlab.xiph.org/xiph/opus.git`, which later resolved into a low-speed timeout. Michael Montalbo suggested workarounds (`http.version=HTTP/1.1`, `http.lowSpeedLimit`), but the **root cause remains unidentified**. The issue may be tied to network constraints (e.g., Anubis proxy).

**`git cat-file --batch-command` remote object-info** -- Pablo Sabater’s **15-patch series** implementing `remote-object-info` for `git cat-file --batch-command` addressed all prior feedback, including **dynamic capability-based placeholder validation** and **memory safety fixes**. Karthik Nayak’s review of patches 10–11 focused on **error-handling philosophy** (silent continuation vs. explicit failure) and **transport-layer integration**, but the series is **technically complete** pending the fix for a critical refactoring flaw in patch 5/13.

**`git log -L` range-scoped diff stat/whitespace/pickaxe** -- Michael Montalbo’s **7-patch v2 series** extending `git log -L` to support `--stat`, `--check`, and `-G` incorporated Junio’s documentation feedback (avoiding "range-scoped" jargon). The series reuses the line-range filter infrastructure and adds **extensive test coverage**, including edge cases at range boundaries. No substantive objections remain.

**Usability improvements for `git branch` and `git push`** -- Harald Nordgren’s **2-patch v3 series** improving error messages for common command-line slips (e.g., `git branch --set-upstream-to=origin main`) was **queued in `next`**. The series introduces config-based advice (`advice.setUpstreamFailure`, `advice.pushRepoLooksLikeRef`) and **thorough test coverage**, addressing all prior feedback.

**Gitk/Git-GUI quiet build alignment** -- Harald Nordgren’s patch aligning gitk and git-gui’s translation catalog generation with core Git’s `make -s` conventions was **merged into Johannes Sixt’s testing branches**. The changes will be proposed for upstream Git inclusion in the coming weeks, though not in time for v2.55.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **2-line patch** fixing a regression in `set_curl_proxy_type()` (where HTTPS proxy URLs were incorrectly rejected) was **merged to `master`**. The fix preserves the original hardening intent while correcting a control-flow error.

---

## On the radar

- **`git repack --geometric --cruft` RFC**: Taylor Blau’s series remains **blocked on the reachability traversal flaw** in `--stdin-packs=follow-reachable`. The proposed fix (reversing traversal order) may require significant rework.
- **`git replay --linearize`**: Toon Claes’s series is **ready for review**, with Junio’s documentation nit as the only open item.
- **`git ls-remote` hang**: The **root cause remains unidentified**, and no similar reports have surfaced. Further investigation may require network-level debugging.