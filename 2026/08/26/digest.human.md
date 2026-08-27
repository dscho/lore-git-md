# Git mailing list daily digest for 2026/08/26

## The day in brief
Git’s global config listing now faces a maintainer debate over Windows path normalization, while a new `receive-report` hook for `git-receive-pack` graduates to `seen` after resolving naming and exit semantics. A reftable optimization series reports a 15-20% performance gain, and Junio C Hamano’s recurring “What’s cooking” report lists 56 topics merged to `master` in the latest batch.

## Notable threads

### Global config listing inconsistency
Delilah Ashley Wu’s three-patch v2 series fixes Git’s long-standing inconsistency where `git config list --global` only showed `$HOME/.gitconfig` while Git actually reads both that and `$XDG_CONFIG_HOME/git/config`. The series is technically complete, with all feedback addressed except for patch 1’s Windows path normalization, which Junio C Hamano questions as extraneous to the core fix.

Junio’s latest feedback [2026/08/26/17-58-57] challenges whether the slash normalization in `path.c` is truly required, suggesting it may be unnecessary for the series’ goal. The patch itself is sound—it modifies `path.c` to apply `convert_slashes()` only on Windows, adds comprehensive test coverage, and avoids broader `cleanup_path()` side effects—but Junio’s skepticism may block its inclusion unless Wu can better justify its relevance.

The series touches `config.c`, `builtin/config.c`, `path.c`, and tests, with no on-disk format changes. The core infrastructure (patches 2 and 3) is uncontroversial, but patch 1’s fate hangs on this debate.

---

### `receive-report` hook for `git-receive-pack`
Karthik Nayak’s three-patch v4 series introduces a new `receive-report` hook for `git-receive-pack`, enabling server administrators to intercept and modify the status report sent to clients after ref updates. The hook is motivated by GitLab’s need to implement MVCC (multi-version concurrency control), where the status report must reflect the repository state after all ref updates are committed but before the client receives confirmation.

The series is now queued in `seen` [2026/08/26/10-19-36] after addressing all prior feedback, including the hook naming objection (renamed from `report` to `receive-report`) and exit status semantics. The hook runs after all ref updates are committed but before the status report is sent, receiving the pkt-line encoded report on stdin and replacing it with its stdout. A non-zero exit status rewrites all ref status lines to `"receive-report hook failed"` and discards the hook’s stdout, matching the behavior of the `pre-receive` hook.

Files touched include `builtin/receive-pack.c`, `Documentation/githooks.adoc`, `Documentation/git-receive-pack.adoc`, and a new test script `t/t5412-receive-report-hook.sh`. The series is technically complete and awaits graduation to `next`.

---

### Reftable stack reload optimization
Karthik Nayak’s four-patch v2 series optimizes reftable stack reloads during transactions, eliminating redundant `fstat()` calls that scale linearly with ref count. Independent benchmarks by Jeff King confirm a 15-20% performance improvement, resolving the earlier 1-2% discrepancy attributed to test methodology differences (`refs/tags/*` vs `refs/heads/*`).

The series is technically complete, with all prior feedback incorporated. Junio C Hamano’s safety concern about premature lock release when multiple `reftable_addition` instances operate on the same stack is addressed in patch 3 via per-addition lock tracking, validated by a new unit test. The only remaining step is Junio’s final assessment [2026/08/26/09-47-06].

Files touched include `refs/reftable-backend.c`, `reftable/stack.h`, `reftable/stack.c`, and `t/unit-tests/u-reftable-stack.c`. The optimization is self-contained and does not affect on-disk formats.

---

### `git worktree add` basename handling
René Scharfe’s four-patch v1 series fixes and cleans up the `worktree_basename()` helper in `git worktree add`, addressing an out-of-bounds read, rejecting malformed paths (e.g., paths consisting solely of separators), and trimming trailing slashes from derived branch names. The series is ready to advance after Junio C Hamano approved the code changes and will apply a minor commit-message tweak locally [2026/08/26/14-35-07].

The patches touch `builtin/worktree.c` and add tests in `t/t2400-worktree-add.sh`. The changes are mechanical and well-scoped, with no behavior changes beyond the fixes. The series supersedes an earlier stalled effort and addresses a real-world issue reported in Git for Windows (issue #6346).

---

### `git commit --amend` during conflict resolution
Elijah Newren’s single-patch bugfix extends Git’s existing protection against `git commit --amend` during conflict resolution to cover `git am`, `git revert`, and all forms of `git rebase`. The patch is well-motivated and addresses a real foot-gun, with clear documentation of the rebase-merge directory files and their roles in distinguishing conflict stops from legitimate amend operations.

Phillip Wood suggested a structural improvement [2026/08/26/13-56-25], proposing to move the conflict-resolution checks from `builtin/commit.c` to `sequencer_determine_whence()` to avoid hardcoded paths. Junio C Hamano endorsed the patch but proposed leaving a `#leftoverbits` marker for future work on `git commit <paths>` [2026/08/26/16-22-42] and suggested a minor cleanup to replace `the_repository` with `s->repo` [2026/08/26/16-39-21].

The patch touches `builtin/commit.c` and adds tests in `t3404-rebase-interactive.sh`, `t3507-cherry-pick-conflict.sh`, and `t4151-am-abort.sh`. It is uncontroversial in its goal and likely to see a v2 incorporating feedback.

---

### `git repo info` path keys
K Jayatheerth’s seven-patch v5 series adds seven new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`) to the `git repo info` command. The series is now build-unblocked after Junio C Hamano requeued it onto a newer base that includes the `repo->prefix` dependency [2026/08/26/14-18-34].

Junio’s latest review [2026/08/26/07-23-06] raises a substantive architectural concern: duplication of logic between `git repo info` and `git rev-parse` for keys like `path.cdup`, `path.toplevel`, `path.superproject-root`, and `path.git-prefix`. He proposes consolidating shared logic into a new helper library (e.g., `repo-info.c`) to avoid long-term maintainability issues. This feedback may require a v6 or follow-up series.

The series touches `builtin/repo.c`, `Documentation/git-repo.adoc`, and `t/t1900-repo-info.sh`. It reuses existing `git log` logic for consistency and maintains backward compatibility.

---

### `git stash` branch-aware design
Vladimir Sitnikov’s RFC proposes a branch-aware or worktree-aware stash design to prevent silent interference between worktrees’ stash operations. Phillip Wood’s follow-up [2026/08/26/10-08-55] advances the design with a reflog-based solution that associates stashes with the specific commit (not branch) they were created on, enabling per-worktree isolation for detached HEAD worktrees without new configuration or ref namespaces.

The proposal uses HEAD’s reflog and the first parent of stash commits to identify the most recent stash for a given commit, avoiding synthetic branch names or configuration knobs. The discussion has shifted from “whether” to “how” to implement stash isolation, with the technical feasibility now well-established. Open questions focus on user interface (e.g., default behavior of `git stash pop`).

---

### `git branch -d` upstream protection
Harald Nordgren’s two-patch v1 series proposes protecting local upstream branches from deletion via `git branch -d`, altering the semantics of the `-d` flag to refuse deletion unless `-D` (force) is used. Junio C Hamano reaffirmed his substantive backward-compatibility objection [2026/08/26/14-37-59], framing the discussion as a balance between offering an “improved” protection feature and preserving the long-standing semantics of `-d`.

Tuomas Ahola clarified [2026/08/26/08-46-41] that Junio’s earlier feedback was a brainstorm of two possible behaviors (blocking deletion or retargeting dependent branches), not an endorsement of the current approach. The core tension remains unresolved: whether the simpler “block deletion” approach is sufficient or whether the patch must implement retargeting to be acceptable.

---

### `git format-rev` formatting options
Kristoffer Haugsbakk’s five-patch v2 series adds `--abbrev`, `--color`, and `--date` options to `git format-rev`, bringing it closer to parity with `git log`. The series is uncontroversial and addresses all v1 feedback, but Junio C Hamano noted it has received no review feedback since posting [2026/08/26/20-33-32] and invited input from contributors more invested in `git format-rev`.

The series touches `builtin/name-rev.c`, `Documentation/git-format-rev.adoc`, and `t/t6120-describe.sh`. It reuses existing `git log` logic for consistency and includes two new refactoring patches (2/5 and 4/5) addressing assertion placement and documentation reuse.

---

### `git symbolic-ref` test coverage
Nikolaus Schuetz’s v3 test patch adds systematic test coverage for `git symbolic-ref`’s exit codes and output behavior when querying a non-symbolic ref, both with and without the `--quiet` (`-q`) flag. The patch is uncontroversial and well-crafted, but the thread’s broader compromise—documenting the current exit code behavior (128 vs. 1) in the man page—is now in question after Junio C Hamano objected to “casting wrong behavior into stone” [2026/08/26/16-51-47].

The patch touches only `t/t1401-symbolic-ref.sh` and uses `test_must_fail` instead of hardcoding exit codes. The unresolved documentation question may delay merging until the thread reaches consensus on how to handle the man page.

---

### `git worktree add` `--guess-remote` ambiguity
Yoichi NAKAYAMA’s four-patch v9 series improves user-facing error messages for ambiguous remote branch names in `git worktree add`, `git checkout`, and `git switch`. The series is now functionally complete and merged to `next`, with all prior feedback addressed, including the `--guess-remote` edge case and organizational refinements.

The final patch [2026/08/26/10-45-10] ensures the `--guess-remote` option in `git worktree add` errors out when multiple matches exist, aligning it with the behavior of `git checkout` and `git switch`. The series touches `builtin/worktree.c`, `builtin/checkout.c`, `checkout.c`, `checkout.h`, and `t/t2400-worktree-add.sh`.

---

### `http.sslVerifyStatus` for OCSP stapling
Graysongordon-gl’s v6 patch adds a boolean `http.sslVerifyStatus` option (default `false`) to enable OCSP staple validation via libcurl’s `CURLOPT_SSL_VERIFYSTATUS`. The patch is technically complete and ready for `next` [2026/08/26/22-01-49], with all prior feedback addressed, including Junio C Hamano’s request for clearer documentation wording.

The patch targets a niche but critical security gap: OpenSSL-linked Git binaries (common in FIPS-compliant deployments) currently ignore OCSP staples, leaving revoked certificates unchecked. When enabled, `http.sslVerifyStatus` enforces fail-closed behavior, aligning with government and enterprise security mandates.

---

### `gitk` AI contribution policy
Johannes Sixt’s documentation patch updates `gitk/README.md` to explicitly discourage AI-generated contributions, aligning with upstream Git’s AI policy. The patch is uncontroversial, but Weijie Yuan’s earlier policy challenge [2026/08/26/06-55-12] has been retracted, and brian m. carlson’s endorsement [2026/08/26/21-55-41] is brief but supportive.

The patch touches only `gitk/README.md` and adds four lines discouraging AI contributions. The discussion has settled on the original scope, with no further policy debate.

---

### `git whatchanged` deprecation feedback
Edvard’s user report [2026/08/26/07-18-54] notes that the deprecation of `git whatchanged` (now requiring `--i-still-use-this`) does not disrupt their workflow, as alternatives like `git log --oneline --name-status` suffice. Kristoffer Haugsbakk’s follow-up [2026/08/26/07-46-38] seeks to clarify what specifically the user values about `whatchanged` (output format vs. command name) and suggests an alias configuration as a potential workaround.

The thread is purely feedback-oriented, with no technical changes proposed.

---

### Outreachy December 2026 cohort
Christian Couder’s administrative announcement [2026/08/26/10-55-19] invites volunteers, project ideas, and feedback on Git’s participation in Outreachy’s December 2026 cohort. Usman Akinyemi, a former Outreachy intern for Git, volunteered to co-mentor again this year [2026/08/26/11-39-39], citing the program’s positive impact.

The thread is focused on community engagement and planning, with no technical changes involved.

---

### Deprecation warning rewording
Junio C Hamano’s v2 documentation patch rewords the user-facing deprecation warning in `usage.c` to eliminate misleading language that could imply the Git project might reverse a deprecation decision. The new message is shorter (6 lines, down from 8), more direct, and guides users to the breaking changes documentation and mailing list archives.

The patch touches only `usage.c` and modifies the error message string in `you_still_use_that()`. The opening line is refined to “will be removed” (previously “is nominated for removal”) to reinforce the finality of deprecation [2026/08/26/17-48-29]. Michael Montalbo’s feedback on phrasing and formatting has been addressed, and the patch is ready for merging.

---

### “What’s cooking” report
Junio C Hamano’s August 2026 (#11) “What’s cooking” report [2026/08/26/23-21-10] lists 56 topics graduated to `master` in the 19th batch since Git 2.55, totaling 563 non-merge commits on `master`. Another 75 topics are cooking in `next`, and 209 are in `seen`.

Key graduated topics include:
- `cc/fast-import-usage`: Standardized `git fast-import` usage and documentation.
- `kk/merge-base-exhaustion`: Optimized merge-base computation by early termination.
- `ps/odb-streams`: Unified object database streaming APIs into a single `struct odb_stream`.
- `ss/repack-drop-filtered`: Added `--drop-filtered` to `git repack` to reclaim space from local promisor blobs.
- `js/packfile-fast-append`: Fixed a performance regression in `packfile_list_append()`.

Notable new topics:
- `dw/config-read-both-global`: Fixes an inconsistency where `git config --global` only read `$HOME/.gitconfig` but not `$XDG_CONFIG_HOME/git/config`.
- `kn/reftable-optimize-reloading`: Optimizes reftable stack reloads by avoiding redundant `stat()` calls.
- `ps/odb-alternates-at-creation`: Defers alternates setup to ODB creation time during clone.
- `ps/odb-pluggable-fsck`: Moves fsck checks into backend-specific layers, making them pluggable.
- `en/no-amend-during-conflicts`: Teaches `commit --amend` to refuse during conflict resolution.

Stalled topics include `ps/libgit-in-subdir`, `tb/pack-with-duplicates`, and `cl/regexec-macos-leak`.

---

### `die_for_incompatible_opts()` helper
Junio C Hamano’s two-patch refactoring series [2026/08/26/23-31-50] introduces `die_for_incompatible_opts()`, a varargs helper replacing the fixed-arity `die_for_incompatible_optN()` family (N=2,3,4). The series is mechanical and well-scoped, touching `parse-options.c`, `parse-options.h`, 14 built-in commands, and the `revision` subsystem.

The first patch reorders parameters of the existing `die_for_incompatible_optN()` functions to enable a NULL sentinel in varargs, and the second patch introduces the new helper and updates all call sites. The fixed-arity helpers remain as thin wrappers around the new varargs function. The change is purely internal API refactoring with no behavior change.

## In brief
- **[2026/05/18/11-22-24]** Junio solicited final review for Erik Cervin-Edin’s v2 2/2 patch completing `--fixup` message specification support, noting it has sat unreviewed.
- **[2026/06/12/20-07-05]** Taylor Blau updated the third patch in his MIDX incremental-writes series, proposing a replacement comment for the outdated block in `midx-write.c` to clarify the relaxed invariant.
- **[2026/06/17/15-30-55]** Junio marked Lutz Lengemann’s zsh completion bugfix patch for inclusion in `next`.
- **[2026/07/12/00-38-26]** Junio pinged Farid Zakaria about the status of the `sequencer: respect --empty for autosquash-induced empty commits` patch, noting no changes since the author’s logistical update.
- **[2026/07/08/02-59-40]** Junio called for review on Michael Montalbo’s v3 series fixing race conditions in HTTP daemon test helpers, noting the absence of feedback from HTTP test experts.
- **[2026/07/06/11-58-14]** Junio pinged Ian Jackson about the timeline for posting v2 of the `git-subtree` safeguard series, noting no response to his earlier procedural follow-up.
- **[2026/08/05/11-53-38]** Junio pinged the thread on Tian Yuchen’s `environment.c` repository configuration refactoring, noting the discussion went dark after an exploratory exchange about decoupling configuration from `the_repository`.
- **[2026/08/11/17-02-00]** No new deltas; the `http.sslVerifyStatus` patch remains ready for `next`.
- **[2026/08/23/09-25-42]** Weijie Yuan retracted their policy challenge on the `gitk` AI contribution patch, and brian m. carlson briefly endorsed the patch.