# The Git Mailing List Daily Digest for 2026/06/27

## The day in brief.

Saturday’s traffic was moderate (50 emails in 15 threads), with a mix of long-running series reaching resolution, a security inquiry that fizzled, and a handful of user reports. The standout was the **`git cat-file --batch-command` remote-object-info series**, which cleared its final documentation nits and is now ready for merging—though a philosophical debate over silent failures versus explicit errors lingers. Elsewhere, **`git replay --linearize`’s post-merge edge case** gained clarity, and **`git repack --geometric --cruft`’s correctness flaw** was diagnosed but not yet fixed.

---

## Notable threads

### `git cat-file --batch-command` remote-object-info series reaches final readiness
The 14th iteration of Pablo Sabater’s GSoC series implementing `git cat-file --batch-command` for remote object metadata queries cleared its last documentation nits. All seven patches now address prior feedback, including Karthik Nayak’s suggestions for clearer placeholder documentation and Junio’s commit-message tweaks. The series introduces `--stdin-packs=follow-reachable` and dynamic format validation, enabling secure, capability-scoped queries for object size and (eventually) type.

The sole unresolved question is **error-handling philosophy**: should the client fail explicitly when metadata is missing, or continue silently (matching local `git cat-file` behavior)? Pablo defends silent continuation as consistent and pragmatic, while Karthik argues it complicates future extensibility. Junio has not weighed in, leaving the decision to the maintainer. With all technical concerns resolved, the series is poised for `next` once the maintainer decides.

---

### `git replay --linearize`’s post-merge edge case clarified
Phillip Wood confirmed Junio’s analysis of a behavioral divergence in `git replay --linearize`: when replaying a merge commit (Z) that merges two divergent branches (X and Y), the command grafts only the first-parent branch (X) onto the new history, leaving Y dangling. This differs from `git rebase --no-rebase-merges`, which replays both branches. Phillip’s reply strengthens the case for follow-up work—either documenting the limitation or adding a `--first-parent=<branch>` option. The series itself remains merged, with no open technical questions.

---

### `git repack --geometric --cruft`’s correctness flaw diagnosed
Taylor Blau’s RFC series combining `--geometric` and `--cruft` repack modes hit a snag: Junio identified a **non-trivial correctness issue** in the two-phase traversal logic of `--stdin-packs=follow-reachable`. The current implementation marks *all* tag objects in included packs as reachable, even if they’re not referenced by any ref, risking unreachable objects in the output pack. Taylor proposed reversing the traversal order (walk from refs first, then mark objects in included packs), but no concrete fix has been submitted. The series remains in RFC limbo until this is resolved.

---

### `git log -L` gains range-scoped diff stat, whitespace checking, and `-G` pickaxe
Karthik Nayak’s seven-patch series extending `git log -L` to support range-scoped `--stat`, `--check`, and `-G` pickaxe operations resurfaced in v2. The only change from v1 was rewording documentation to avoid "range-scoped" jargon, addressing Junio’s feedback. The series is now complete, with thorough test coverage for edge cases (root commits, renames, multiple ranges). No substantive reviews have been posted yet, but the implementation is clean and uncontroversial.

---

### `excludes_file` libification patch sparks guardrail debate
Tian Yuchen’s patch moving the global `excludes_file` variable into `struct repo_config_values` as part of the libification effort prompted a micro-debate over defensive programming. Junio proposed replacing the silent `repo->initialized` early return in the getter with a `BUG()` assertion, arguing that all callers should already be working with an initialized repository. Tian’s patch is otherwise ready for `next`, with all prior feedback addressed. The discussion reflects the project’s preference for fail-fast design in foundational refactoring work.

---

## In brief

**`git branch` and `git push` usability improvements** -- Harald Nordgren’s v3 series improving error messages for common command-line slips (e.g., `git branch --set-upstream-to=origin main`) is queued in `seen` and slated for `next`. The series adds config-based advice for missing slashes and slash-as-space misformatting, with thorough test coverage.

**`gitk` and `git-gui` quiet build alignment** -- Johannes Sixt confirmed that the changes to silence translation-catalog build output during `make -s` are in his testing branches and will be proposed for upstream Git inclusion in the coming weeks. The patches align gitk and git-gui with core Git’s quiet build conventions.

**`git ls-remote` hang resolved** -- Steffen Nurpmeso’s report of a 100% CPU hang in `git ls-remote` against `https://gitlab.xiph.org/xiph/opus.git` disappeared without code changes, likely due to external factors (e.g., Anubis proxy rate-limiting). No root cause was identified.

**OpenSSL CVE inquiry closed** -- Todd Zullinger clarified that CVE-2026-34182 (affecting OpenSSL 3.5.6) does not impact Git, as the vulnerability is in CMS functionality not used by Git’s TLS stack. The thread closed with no action required.

**HTTP proxy regression fixed** -- Johannes Schindelin’s patch fixing a regression in `663d7abe07ea` (which incorrectly rejected HTTPS proxy URLs) is a minimal, self-contained fix to `http.c`. The patch mirrors the early returns used for HTTP and SOCKS proxies.

**`greplint.pl`’s automation risks highlighted** -- SZEDER Gábor identified a pre-existing bug in `t3420-rebase-autostash.sh` that Michael Montalbo’s `greplint.pl` series would have preserved. Junio proposed extending `test_grep` to catch such cases, offering a path to reconcile automation with diagnostic rigor.

**Source tree reorganization’s trade-offs debated** -- SZEDER Gábor raised a **user-facing concern** about Patrick Steinhardt’s RFC to move `libgit.a` components into a `lib/` directory: the reorganization disrupts `git log --follow` and exacerbates the "rename barrier" problem. The objection shifts the debate from implementation details to whether the benefits (structural clarity) justify the costs (historical tracking difficulties).

**`t3420-rebase-autostash.sh` fragile test fixed** -- SZEDER Gábor’s five-year-old patch resurfaced and gained Phillip Wood’s endorsement. The fix replaces a fragile `! grep dirty file3` check with `test_path_is_missing file3`, correctly asserting the file’s absence after `git rebase --quit`. The patch is ready for merging once the commit message typo ("Thighten" → "Tighten") is corrected.

---

## On the radar

**`git repack --geometric --cruft`’s correctness fix** -- Taylor Blau’s proposed traversal-order reversal for `--stdin-packs=follow-reachable` needs implementation. The flaw risks retaining unreachable objects, undermining the series’ core goal.

**`git replay --linearize`’s merge-commit divergence** -- Junio and Phillip Wood’s analysis of the post-merge edge case may prompt follow-up work to document the behavior or add user control (e.g., `--first-parent=<branch>`).