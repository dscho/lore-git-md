# The Git Mailing List Daily Digest for 2026/06/27

**The day in brief.** Saturday’s traffic (50 emails in 15 threads) was a mix of late-week cleanup and lingering RFCs. The standout was **Taylor Blau’s RFC combining `--geometric` and `--cruft` repacks**, which hit a **critical correctness snag** in its reachability filtering logic. Elsewhere, **Tian Yuchen’s libification patch** for `excludes_file` cleared its last review hurdle, and **Harald Nordgren’s usability series** for `git branch` and `git push` typos landed in `next`. A **security inquiry about OpenSSL CVE-2026-34182** fizzled after it became clear Git isn’t exposed.

---

## Notable threads

### **`git repack --geometric --cruft`: Correctness flaw in reachability filtering**
Taylor Blau’s 10-patch RFC (v1) to combine `--geometric` and `--cruft` repack modes hit a **non-trivial correctness issue** in patch 8/10. The `--stdin-packs=follow-reachable` logic, designed to ensure only reachable objects from rolled-up packs are included in the geometric pack, **incorrectly retains unreachable tags and objects** by marking *all* tag objects in included packs without verifying reachability from refs. Junio C Hamano’s review flagged the flaw, and Taylor acknowledged it stems from the two-phase traversal’s order: the current implementation first marks objects in included packs, then walks from reference tips, which risks including unreachable objects. Taylor proposed reversing the traversal order to prioritize reachability from refs, but no concrete fix has been submitted yet. The series remains **blocked on this issue**, though patches 1–7 and 9–10 are uncontroversial refactorings. The flaw undermines the series’ core goal of cleanly separating reachable and unreachable objects, and its resolution will likely require significant rework of the traversal logic.

---

### **`git replay --linearize`: Behavioral clarification and test coverage**
Phillip Wood joined the `git replay --linearize` thread to clarify the **algorithmic difference** between `git replay --linearize` and `git rebase --no-rebase-merges` when flattening a history with divergent merges. Where rebase rewrites both branches and drops the merge (producing `A->X'->Y'` or `A->Y'->X'`), replay drops the merge and keeps only one branch (e.g., `A->X'` with `Y'` lost), leaving the other branch dangling. The discussion highlighted a **user-expectation mismatch**, and Junio proposed a test to lock in the current semantics. The test, which verifies that only one of `X` or `Y` survives after `--linearize`, is likely to be incorporated in the next iteration. The thread remains **technically complete** (v6 addressed all prior feedback), but this new discussion may delay merging until the behavioral clarification is resolved. The series’ divergence from `git rebase`’s syntax (`--linearize` vs. `--rebase-merges=<mode>`) is now explicitly justified in the commit message, meeting Patrick Steinhardt’s earlier requirement.

---

### **`git cat-file --batch-command`: Remote object-info series nears completion**
Pablo Sabater’s **15-patch series** implementing `git cat-file --batch-command` for remote object metadata queries is now **technically complete** after addressing all prior feedback. The series introduces the `remote-object-info` command, dynamic capability-based placeholder validation, and comprehensive memory safety improvements. Karthik Nayak’s **substantive reviews** of patches 10/13 and 11/13 focused on **error handling and transport layer integration**, with Pablo accepting all suggestions. The only remaining debate—whether the client should fail explicitly or continue silently when metadata is missing—is now **documented** and no longer blocking. The series is **ready for merging pending the fix to patch 5/13** (a critical refactoring flaw identified by Junio), which is straightforward to address. The implementation includes **680 lines of new tests** and verified memory safety, making it a strong candidate for `next`.

---

### **`excludes_file` libification: Final review hurdle cleared**
Tian Yuchen’s patch to move the global `excludes_file` variable into `struct repo_config_values` cleared its last review hurdle after Junio C Hamano’s **substantive critique** of the defensive programming style in the getter. The patch now uses a stricter `repo->initialized` check instead of a silent early return, aligning with the project’s preference for fail-fast design in foundational refactoring work. The series is **merged into `next`**, completing the **three-phase migration path** (silent return → `BUG()` → no check) proposed by Christian Couder. The patch is a **clean, incremental step** in the `the_repository` removal effort, with no user-facing behavior changes. The destructor’s interaction with other repository cleanup paths remains a point of potential discussion, but the patch is uncontroversial and well-aligned with libification goals.

---

### **Usability improvements for `git branch` and `git push` typos**
Harald Nordgren’s **two-patch series** improving error messages for common command-line slips (`git branch --set-upstream-to=origin main` and `git push origin/main`) is now **merged into `next`**. The series introduces new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and helper functions to detect and suggest corrections for these mistakes. Junio’s feedback on the redundant advice check in the first patch was addressed by documenting it as intentional, and the series now includes thorough test coverage. The implementation is **clean and focused**, addressing a clear usability pain point without disrupting existing functionality. The series could serve as a pattern for similar improvements in other commands (e.g., `git fetch`), though no follow-up work is planned yet.

---

## In brief

**`git log -L` range-scoped diff stat, whitespace check, and `-G` pickaxe** -- Michael Montalbo posted v2 of a seven-patch series extending `git log -L` to support `--stat`, `--check`, and `-G` with tracked line ranges. The series reuses the line-range filter infrastructure to scope stat counts, whitespace errors, and pickaxe matches to the specified ranges. Junio’s feedback on documentation phrasing was incorporated, and the series is now ready for further review.

**`greplint.pl` test suite conversion** -- Michael Montalbo’s series converting bare `grep` assertions to `test_grep` in the test suite is **approved for merging** after addressing all prior feedback. The series found **10+ pre-existing bugs** during conversion, including a latent lexer bug in `lib-shell-parser.pl`. Junio proposed extending `test_grep` to catch "file missing" scenarios, which could further improve the linter’s safety.

**`git ls-remote` hang investigation** -- Steffen Nurpmeso reported a transient infinite loop in `git ls-remote` against `https://gitlab.xiph.org/xiph/opus.git`, which later resolved into a low-speed timeout. Michael Montalbo suggested work-around settings (`http.version=HTTP/1.1`, `http.lowSpeedLimit`, `http.lowSpeedTime`), but the root cause inside Git remains unidentified. The thread is now closed from a user-support standpoint.

**OpenSSL CVE-2026-34182 inquiry** -- A security inquiry about OpenSSL CVE-2026-34182 (affecting OpenSSL 3.5.6 bundled with Git 2.54.0) fizzled after Todd Zullinger and Johannes Schindelin clarified that Git’s OpenSSL usage does not invoke the CMS functionality affected by the CVE. The thread is now closed, with no Git release or advisory planned.

**`gitk` and `git-gui` quiet build output** -- Harald Nordgren’s series to silence translation-catalog build output during `make -s` for `gitk` and `git-gui` is now **merged into Johannes Sixt’s testing branches**. The changes align with core Git’s quiet build conventions and will be proposed for upstream Git inclusion in the coming weeks.

**`git replay --linearize` regression fix** -- Toon Claes’s `git replay --linearize` series (v6) fixed a **regression** where replaying a single branch containing merge commits would drop commits. The fix restored the `replayed_base` parameter to ensure all replayed commits are flattened into a single topology. The series is **technically complete** and ready for review.

**Test fix for `git rebase --quit`** -- SZEDER Gábor resurfaced a five-year-old patch fixing a fragile test in `t/t3420-rebase-autostash.sh`. The patch replaces `! grep dirty file3` with `test_path_is_missing file3` to correctly assert that `file3` does not exist after `git rebase --quit`. The patch is **queued in `next`** pending a commit message typo fix ("Thighten" → "Tighten").

---

## On the radar

**`git repack --geometric --cruft` correctness issue** -- Taylor Blau’s RFC remains blocked on the reachability filtering flaw in `--stdin-packs=follow-reachable`. The proposed fix (reversing the traversal order) is non-trivial and may require significant rework. Downstream consumers should watch for updates, as the series’ core functionality hinges on this issue.

**`git replay --linearize` behavioral clarification** -- The discussion about the algorithmic difference between `git replay --linearize` and `git rebase --no-rebase-merges` may delay merging until the behavioral clarification is resolved. The proposed test to lock in the current semantics is likely to be incorporated in the next iteration.