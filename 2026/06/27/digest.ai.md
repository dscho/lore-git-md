## The day in brief

Saturday, June 27, 2026 was a **moderate-traffic day** (50 emails, 15 threads) with a **user-support and refactoring focus**. The standout event was a **security inquiry** about CVE-2026-34182 in OpenSSL 3.5.6, which was quickly resolved after clarifying that Git’s usage does not expose users to the vulnerability. The **`git cat-file --batch-command` remote object-info series** saw its final documentation nits addressed, positioning it for imminent merging. Meanwhile, **libification efforts** continued with multiple patches moving global variables into repository-scoped structures, though one thread revealed a **correctness debate** about error-handling philosophy. A **user-reported hang** in `git ls-remote` resurfaced briefly but remains unresolved at the code level.

---

## Notable threads

### Security inquiry: CVE-2026-34182 (OpenSSL 3.5.6 in Git 2.54.0)
Tim Person opened the day with an urgent inquiry about a **critical CVE** in OpenSSL 3.5.6, bundled with Git 2.54.0, asking when an updated installer would be available. **Todd Zullinger** clarified that the Git project does not distribute binaries (the Windows installer is handled by Git for Windows) and that Git’s OpenSSL usage does not invoke the CMS functionality affected by the CVE. **Johannes Schindelin** confirmed that the OpenSSL executable bundled with Git for Windows is not critical and will be updated in the next release (v2.55.0, June 29–30). The reporter formally accepted the assessment, closing the thread. **No Git release or advisory is planned**, as the project is not exposed to the vulnerability.

---

### `git cat-file --batch-command` remote object-info series nears completion
Pablo Sabater’s **15-iteration series** implementing `remote-object-info` for `git cat-file --batch-command` saw its final documentation nits addressed in today’s traffic. The series, which enables querying object metadata from remotes without full downloads, is now **technically complete** after resolving all prior feedback, including **dynamic capability-based placeholder validation**, **memory safety improvements**, and **protocol extensibility consensus**. The only remaining action is a **critical refactoring fix** in patch 5/13 (uninitialized variable and lingering global variable), which Pablo has acknowledged and is expected to address shortly. The series is **poised for merging** once this fix is implemented, marking a significant milestone in Git’s remote query capabilities.

---

### Libification effort: `excludes_file` migration sparks guardrail debate
Tian Yuchen’s patch moving the global `excludes_file` variable into `struct repo_config_values` as part of the libification effort **drew a substantive review** from Junio C Hamano. The debate centered on the **error-handling philosophy** of the new getter function: Junio argued for a `BUG()` assertion to catch callers accessing uninitialized repositories, while Tian initially favored a silent early return. The discussion reflects broader tensions in the libification effort between **fail-fast design** and **backward compatibility**. The patch is otherwise **technically sound**, with all prior feedback addressed, and is expected to land once the guardrail debate is resolved. This thread highlights the **challenges of eliminating global state** while maintaining robustness.

---

### `git replay --linearize` series clarifies behavioral difference from `git rebase`
Phillip Wood joined the `git replay --linearize` thread to clarify a **key behavioral difference** from `git rebase --no-rebase-merges`. Where rebase rewrites both branches of a divergent merge (e.g., `A->X'->Y'`), replay drops the merge and keeps only one branch (e.g., `A->X'`), leaving the other dangling. The discussion reinforced the need for **documentation and test coverage** of this edge case, which was added in v6. The series is **technically complete** and addresses all prior feedback, but this clarification may delay merging until the behavioral implications are fully locked in. The thread underscores the **importance of precise semantics** in new Git commands.

---

### `git ls-remote` hang resurfaces, remains unresolved
Steffen Nurpmeso reported that `git ls-remote https://gitlab.xiph.org/xiph/opus.git` initially entered a **100% CPU busy-loop**, then later exhibited a **low-speed timeout** under constrained network conditions. **Michael Montalbo** provided troubleshooting suggestions (e.g., `GIT_TRACE_CURL=1`, `http.lowSpeedLimit`), but the root cause inside Git remains unidentified. The original busy-loop symptom has not recurred, and no similar reports have emerged, leaving the issue in a **diagnostic limbo**. The thread highlights the **challenges of debugging transient network-layer issues** in Git’s transport code.

---

## In brief

**`git log -L` range-scoped diff stat, whitespace check, and `-G` pickaxe** -- Michael Montalbo posted v2 of a seven-patch series extending `git log -L` to support `--stat`, `--check`, and `-G` with range scoping. The series addresses prior feedback (documentation phrasing) and is now **ready for review**, with thorough test coverage for edge cases like root commits and renames.

**`greplint.pl` test suite conversion** -- Junio C Hamano proposed extending `test_grep` to detect missing files in negated assertions, addressing a pre-existing bug in `t3420-rebase-autostash.sh`. The discussion highlights the **tension between automated refactoring and latent test bugs**, with no resolution yet.

**Usability improvements for `git branch` and `git push`** -- Harald Nordgren’s v3 series improving error messages for common command-line slips (e.g., `git branch --set-upstream-to=origin main`) is **queued in `seen`** and slated for `next`. The series adds config-based advice (`advice.setUpstreamFailure`, `advice.pushRepoLooksLikeRef`) and is uncontroversial.

**Gitk and git-gui quiet build integration** -- Johannes Sixt confirmed that the changes to silence translation-catalog build output during `make -s` are in his testing branches and will be proposed for upstream Git inclusion in the coming weeks. The thread is **procedurally closed**.

**`git repack --geometric --cruft` RFC series** -- Taylor Blau’s RFC combining geometric and cruft repack modes **hit a correctness snag** in the `--stdin-packs=follow-reachable` logic, which may retain unreachable tags. The flaw requires **significant rework** of the two-phase traversal, delaying the series’ progress.

**HTTP proxy regression fix** -- Johannes Schindelin’s two-line patch fixing a regression in `set_curl_proxy_type()` (incorrectly rejecting HTTPS proxy URLs) was **merged into `master`**. The fix is minimal and self-contained, addressing a logic error in the HTTP transport layer.

---

## On the radar

- **`git replay --linearize`** -- The behavioral clarification about divergent merges may prompt additional review before merging.
- **`git repack --geometric --cruft`** -- The correctness issue in `--stdin-packs=follow-reachable` requires rework and may delay the series.
- **`git ls-remote` hang** -- No progress on the root cause; may resurface if the issue recurs under similar network conditions.