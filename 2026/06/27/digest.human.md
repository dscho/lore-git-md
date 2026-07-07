## The day in brief

Saturday, June 27, 2026 was a **moderately busy day** on the Git mailing list, with **50 emails across 15 threads**. The standout developments were **two long-running series reaching technical completion**—Harald Nordgren’s usability improvements for misformatted remote/branch arguments, and Michael Montalbo’s `greplint.pl` test-suite linter—both now **merged into `next`**. A **security inquiry** about OpenSSL CVE-2026-34182 was **resolved as non-applicable** to Git, and a **regression fix** for HTTPS proxy URLs was **merged into `master`**. The day also saw **substantive review feedback** on Taylor Blau’s RFC combining `--geometric` and `--cruft` repack modes, and **new bug reports** about `git ls-remote` hangs and a potential infinite loop.

---

## Notable threads

### `git branch` and `git push` usability improvements now merged
**Harald Nordgren’s two-patch series** (v3) improving error messages for common command-line slips—`git branch --set-upstream-to=origin main` (missing slash) and `git push origin/main` (slash instead of space)—**landed in `next`**. The patches add new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and helper functions to detect these mistakes and suggest corrections. Junio C Hamano queued the series after addressing all feedback, including test style fixes and documentation of a redundant advice check. The implementation is **non-disruptive**, focused solely on error messaging, and directly addresses a frequent user pain point. **Files touched**: `builtin/branch.c`, `builtin/push.c`, `advice.{c,h}`, `Documentation/config/advice.adoc`.

---

### `greplint.pl` test-suite linter series approved for merging
Michael Montalbo’s **six-patch series** (v4) introducing `greplint.pl`, a linter that converts bare `grep` assertions in the test suite to `test_grep` for improved debuggability, **received maintainer approval** and is now **ready for `next`**. The series found and fixed **10+ pre-existing test bugs**, including a latent lexer bug in the shared shell parser. Junio C Hamano’s only remaining concern—a pre-existing bug in `t3420-rebase-autostash.sh` preserved by the linter—was addressed by proposing a **technical extension to `test_grep`** to detect missing-file cases. The series is **technically complete**, with all prior feedback incorporated, and represents a **significant step forward in test suite hygiene**. **Files touched**: `greplint.pl`, `greplint-cat.pl`, `lib-shell-parser.pl`, `Makefile`, `t/test-lib-functions.sh`, `t/README`, and ~340 test scripts.

---

### HTTPS proxy regression fix merged
Johannes Schindelin’s **standalone patch** fixing a regression in commit `663d7abe07ea` (http: reject unsupported proxy URL schemes) was **merged into `master`**. The patch corrects a control-flow error in `set_curl_proxy_type()` where the HTTPS branch fell through to a `return -1` intended for unknown schemes, causing Git to incorrectly reject HTTPS proxy URLs. The fix is **minimal (two lines added to `http.c`)** and preserves the original hardening intent. Junio C Hamano initially suggested refactoring the function to return `void` but retracted the idea after recognizing the need to reject unsupported proxy schemes. The patch is **self-contained and uncontroversial**, addressing a specific logic error without introducing new behavior.

---

### OpenSSL CVE-2026-34182 inquiry resolved
A **security inquiry** about CVE-2026-34182 (OpenSSL 3.5.6) in Git 2.54.0 was **resolved as non-applicable**. Todd Zullinger clarified that Git’s OpenSSL usage does not invoke the CMS functionality affected by the CVE, and Johannes Schindelin confirmed that the OpenSSL executable bundled with Git for Windows is not critical. The reporter, Tim Person, formally accepted the assessment, and no Git release or advisory is planned. The next Git for Windows release (v2.55.0, June 29–30) will include OpenSSL 3.5.7.

---

### `git repack --geometric --cruft` RFC faces correctness challenge
Taylor Blau’s **10-patch RFC** combining `--geometric` and `--cruft` repack modes received **substantive review feedback** from Junio C Hamano. The core issue is a **correctness flaw** in the two-phase traversal logic of `--stdin-packs=follow-reachable`, which may retain unreachable tags and objects. Taylor acknowledged the problem and proposed reversing the traversal order to prioritize reachability from refs, but no concrete implementation has been submitted yet. The series also includes a **minor efficiency improvement** (rewriting `pack_geometry_contains_pack()` using `skip_prefix()`) and a **pre-existing quirk** in MIDX retention logic (not addressed). The RFC remains in **early review**, with significant rework expected for the reachability filtering.

---

## In brief

**`git replay --linearize` behavioral clarification** -- Phillip Wood clarified the algorithmic difference between `git replay --linearize` and `git rebase --no-rebase-merges` when flattening divergent merges. The discussion reinforced the need for documentation and test coverage of this edge case, but the series itself remains **technically complete**.

**`git log -L` range-scoped diff stat, whitespace check, and `-G` pickaxe (v2)** -- Michael Montalbo submitted the second iteration of a seven-patch series extending `git log -L` to support range-scoped diff stat formats (`--stat`, `--numstat`, `--shortstat`), whitespace checking (`--check`), and the `-G` pickaxe. The v2 changes are **documentation-only**, addressing Junio’s feedback to avoid "range-scoped" jargon. The series is **ready for review**, with no outstanding technical objections.

**`excludes_file` libification (v3–v6)** -- Tian Yuchen’s refactoring patch moving the global `excludes_file` variable into `struct repo_config_values` **evolved through four iterations**, addressing feedback on guard design, memory management, and patch structure. The series is now **merged into `next`**, completing the three-phase migration path (silent return → `BUG()` → no check) proposed by Christian Couder.

**`git ls-remote` hangs and timeouts** -- Two separate reports emerged about `git ls-remote` misbehaving against specific repositories. One case (Steffen Nurpmeso) initially exhibited a **100% CPU busy-loop**, which later morphed into a **low-speed timeout** under constrained network conditions. The root cause remains **unidentified**, but work-around settings (`http.version=HTTP/1.1`, `http.lowSpeedLimit`, `http.lowSpeedTime`) were suggested. No patch or merge has been produced.

---

## On the radar

**`git repack --geometric --cruft` RFC** -- Taylor Blau’s series combining `--geometric` and `--cruft` repack modes faces a **non-trivial correctness issue** in the reachability filtering logic. The proposed fix (reversing the traversal order) may require significant rework, and the series’ complexity suggests it will need **additional iterations** before merging.

**`git ls-remote` hangs** -- The two reports of `git ls-remote` misbehaving (busy-loop and timeout) remain **unresolved at the code level**. No similar reports have surfaced, but the constrained network conditions suggest a potential **edge case in the HTTP transport layer** worth monitoring.