# Git mailing list daily digest for 2026/08/26

## The day in brief
The Git mailing list saw 68 emails today, with key developments including Junio C Hamano rejecting a patch for `git apply` due to intentional design, Taylor Blau proposing a commit-message tweak for MIDX incremental writes, and Junio marking several topics for integration. Outreachy participation for December 2026 gained momentum with new volunteers, while discussions on stash isolation and deprecation warnings continued to refine user-facing behavior.

## Notable threads

### Global config file handling inconsistency
[2025/10/10/01-14-05]
Delilah Ashley Wu’s three-patch v2 series fixes Git’s global config file handling inconsistency where `git config list --global` only showed `$HOME/.gitconfig` while Git actually reads both that and `$XDG_CONFIG_HOME/git/config`. Today, Junio C Hamano questioned whether the Windows path normalization in patch 1 is truly required for the core fix, calling it potentially extraneous. The series remains technically complete with all other feedback addressed, including structural refinements to patch 3 suggested by Junio.

The series modifies `config_sequence()` in `config.c` to handle global scope by reading both config files through `do_git_config_sequence()`, adding flags to restrict config reading to global scope. The implementation reuses existing infrastructure and maintains backward compatibility. The Windows path normalization (patch 1) remains the only unresolved discussion point, with Junio suggesting it may be unnecessary for the series’ core goal.

### MIDX incremental writes with custom base layers
[2026/06/12/20-07-05]
Taylor Blau’s three-patch series addressing MIDX incremental writes with custom base layers saw progress today. Taylor proposed a replacement comment for the outdated block in `midx-write.c`, explicitly stating the relaxed invariant that `ctx->base_midx` and `ctx->to_include` can both be non-NULL. This addresses Junio C Hamano’s earlier request for clearer documentation of the relaxed invariant when using `--stdin-packs --incremental --base=<foo>`.

The series fixes a bug where the MIDX write path was not properly handling custom base layers during incremental writes, leading to incorrect reachability closure for bitmaps. The fix is specific to the `--stdin-packs` path and does not affect normal MIDX writes. All test cases now pass, including previously failing scenarios for `--base=none` and `--base=<hash>`. The series is ready for integration pending this final documentation update.

### `git apply` memory leak and state corruption
[2026/07/02/04-17-59]
Junio C Hamano rejected a patch that attempted to fix a memory leak and state corruption in `git apply`'s `find_header()` function. The maintainer stated that the current behavior—where abandoned Git-style headers corrupt the real patch—is intentional, serving as a "sanity check" to force users to inspect and fix malformed patches. The rejection is conceptual, with no alternative solutions proposed.

The patch had introduced a temporary `struct patch` to isolate Git-header parsing, only committing state if the header was accepted. Junio’s rejection does not engage with the technical implementation or test coverage, focusing instead on the higher-level design intent. The thread is now effectively closed unless the author provides a rebuttal or revised approach that preserves the intended design.

### `git add -e` refactoring
[2026/07/09/19-26-19]
Junio C Hamano marked Gatla Vishwahwar Reddy’s refactoring patch for `git add -e` for inclusion in the `next` branch. The patch replaces the subprocess call to `git apply` with a direct invocation of Git’s internal `apply_all_patches()` API, eliminating fork overhead and improving code clarity. Junio framed the merge as a low-risk experiment, noting the lack of feedback from users of the `git add -e` feature.

The patch is small and correct, with all prior feedback addressed, including a subdirectory correctness fix by passing `NULL` as the prefix argument to `init_apply_state()`. A new test in `t/t3702-add-edit.sh` verifies subdirectory operation. The change touches `builtin/add.c` and removes dependencies on `run-command.h` and `strvec.h` while adding `apply.h`. The implementation reuses existing `apply_all_patches()` and `apply_state` structs without introducing new symbols.

### `git worktree add` ambiguous remote branch names
[2026/08/08/08-21-41]
Yoichi NAKAYAMA’s four-patch v9 series improving error messages for ambiguous remote branch names in `git worktree add`, `git checkout`, and `git switch` saw final refinements today. The series is now functionally complete, with all prior feedback addressed, including the `--guess-remote` edge case and organizational improvements. Junio C Hamano provided surface-level review feedback on the first two patches, suggesting minor adjustments to commit message clarity and code structure.

The series replaces the terse "matched multiple remote tracking branches" warning with a detailed, actionable advice message that lists conflicting remotes and suggests workarounds like `--track` or setting `checkout.defaultRemote`. The advice is controlled by the new `advice.checkoutAmbiguousRemoteBranchName` config key (enabled by default). The implementation extends `unique_tracking_name()` to collect remote names and introduces a shared `advise_disambiguating_remotes()` helper to avoid code duplication. The series is merged to `next` and ready for `master`.

### Outreachy December 2026 participation
[2026/08/26/10-55-19]
Christian Couder invited volunteers, project ideas, and feedback on Git’s participation in Outreachy’s December 2026 cohort. The deadline for mentoring organizations to sign up is September 11, 2026. Usman Akinyemi, a former Outreachy intern for Git, volunteered to co-mentor again this year, citing the program’s positive impact. Last year, Git mentored one intern funded directly by the project after GitHub and GitLab declined sponsorship.

The thread is administrative, seeking community input on participation, mentors, and project ideas. No technical changes are proposed, but the discussion highlights the program’s value in onboarding new contributors and fostering long-term engagement. Usman’s involvement as a co-mentor strengthens the case for Git’s continued participation.

### Stash isolation for worktrees
[2026/08/18/07-55-55]
Phillip Wood proposed a concrete technical solution to make Git’s stash stack branch-aware or worktree-aware, addressing the detached HEAD gap identified in the previous discussion. The solution uses the reflog of HEAD and the first parent of stash commits to associate stashes with the specific commit (not branch) they were created on, enabling per-worktree isolation without synthetic branch names or configuration knobs.

The proposal focuses on extending `git stash pop` with options to pop the most recent stash from the current branch *or* the current worktree, preserving the global stash stack for users who rely on its current behavior. This hybrid design unifies the branch-aware and worktree-aware approaches under a single mechanism, allowing users to choose the scope of stash isolation at operation time. The solution leverages existing Git machinery (reflogs and commit parent pointers) and avoids new ref namespaces or configuration.

### Deprecation warning rewording
[2026/08/26/14-33-31]
Junio C Hamano posted v2 of a documentation patch rewording the deprecation warning in `usage.c` to eliminate misleading language that could imply the Git project might reverse a deprecation decision. The new message is shorter (6 lines, down from 8), more direct, and guides users to the breaking changes documentation, mailing list archives, and the mailing list for help. Michael Montalbo provided surface-level review feedback, suggesting minor phrasing tweaks, but Junio rejected them, explaining the omission of "replacement" is intentional to cover cases where a deprecated command may not have a direct replacement.

The patch changes the opening line from "this command is nominated for removal" to "this command will be removed" to reinforce the finality of deprecation. The change is purely textual, touching only the error message string in `you_still_use_that()` in `usage.c`. The thread has resolved all formatting questions, and the patch is ready for integration.

## In brief

- **[2026/05/18/11-22-24]** Junio C Hamano solicited final review for Erik Cervin-Edin’s v2 patch completing `--fixup` message specification support, noting it has sat unreviewed.
- **[2026/06/17/15-30-55]** Junio C Hamano marked Lutz Lengemann’s zsh completion bugfix patch for inclusion in the `next` branch, signaling readiness for integration testing.
- **[2026/07/08/02-59-40]** Junio C Hamano noted the HTTP daemon test helper race condition series has seen no review from contributors with HTTP test expertise, despite being technically sound.
- **[2026/07/12/00-38-26]** Junio C Hamano followed up on Farid Zakaria’s `sequencer: respect --empty for autosquash-induced empty commits` patch, asking for status updates.
- **[2026/07/16/01-21-31]** Junio C Hamano queued K Jayatheerth’s `git repo info` path-keys series on top of a newer base to resolve a build-breakage due to the missing `repo->prefix` dependency.
- **[2026/07/25/15-34-27]** Junio C Hamano reached out to Phillip Wood for review on Harald Nordgren’s `git checkout -m` autostash conflict handling series, citing Phillip’s earlier "leftoverbits" comment.
- **[2026/08/05/11-53-38]** Junio C Hamano pinged the `environment.c` repository configuration refactoring thread, noting it has gone quiet after an exploratory discussion about decoupling configuration from `the_repository`.
- **[2026/08/11/17-02-00]** Junio C Hamano’s final call for review on the `http.sslVerifyStatus` feature patch confirmed it is technically complete and ready for `next`.
- **[2026/08/13/21-12-33]** Nikolaus Schuetz posted v3 of a test patch for `git symbolic-ref`, deferring the contested documentation update and focusing on test coverage.
- **[2026/08/19/13-19-36]** Karthik Nayak posted v4 of the `receive-report` hook series, renaming the hook from `report` to `receive-report` to address Junio and Patrick Steinhardt’s naming objection.
- **[2026/08/23/09-25-42]** Weijie Yuan retracted their earlier policy challenge to the `gitk` AI contribution documentation patch, and brian m. carlson endorsed the patch.
- **[2026/08/25/18-03-46]** Junio C Hamano reaffirmed his backward-compatibility concern about Harald Nordgren’s branch deletion protection series, shifting the burden to the author to propose a version that addresses it.
- **[2026/08/26/05-21-21]** Phillip Wood suggested moving conflict-resolution checks from `builtin/commit.c` to `sequencer_determine_whence()` in Elijah Newren’s `git commit --amend` protection patch.
- **[2026/08/26/23-31-50]** Junio C Hamano posted a two-patch refactoring series introducing `die_for_incompatible_opts()`, a varargs helper replacing the fixed-arity `die_for_incompatible_optN()` family.