# Git mailing list daily digest for 2026/09/04

## The day in brief
The Git project saw significant progress on several fronts today. The `receive-report` hook series reached v7, addressing all prior feedback and rebasing onto `master`. A new `--force-if-includes` bugfix series was posted, fixing a long-standing flaw in reflog checking. The `git maintenance` heuristic for `rerere gc` received final endorsements, and a major refactoring series to unify "ref storage" terminology was introduced. Several regression reports were resolved, including one identifying CrowdStrike Falcon as the root cause of macOS-specific Git issues.

## Notable threads

### receive-pack: add new `receive-report` hook
**[2026/09/04/21-28-48 by Karthik Nayak]**

Karthik Nayak posted v7 of the four-patch series introducing the `receive-report` hook for `git-receive-pack`. This hook addresses a server-side extensibility gap by allowing administrators to intercept and modify the pkt-line encoded status report sent to clients after ref updates are committed but before the response is finalized.

The series is now feature-complete, with all prior feedback from Junio C Hamano, Patrick Steinhardt, and others addressed. Key improvements in v7 include:
- Unified report generation by merging `report()` and `report_v2()` into a single helper
- Code style improvements (replacing `if/else` with `switch` statements)
- Rebase onto `master` (3cb9185f65) resolving conflicts with Justin Tobler's concurrent "pluggable writes" series

The hook is motivated by GitLab's need to implement multi-version concurrency control (MVCC), where the final status must reflect operations occurring after `ref_transaction_commit()`. The design intentionally decouples the reported status from the actual repository state—a trade-off explicitly acknowledged in the commit message. This allows the server to report a push as failed (`ng`) while still committing ref updates, which violates Git's traditional consistency guarantees but enables conservative client-side behavior.

The implementation adds 224 lines of test coverage in `t/t5412-receive-report-hook.sh` and updates documentation in `githooks.adoc` and `git-receive-pack.adoc`. The hook is invoked after all ref updates are committed, receives the complete status report on stdin, and replaces it with its stdout; stderr is forwarded to the client via sideband. A non-zero exit code discards the hook's stdout and rewrites all ref status lines to `"receive-report hook failed"`.

The series is queued in `seen` and has no outstanding technical objections. Junio's earlier consistency concern about the hook's ability to report a state divergent from the actual repository remains a documented design choice rather than a blocker.

### push: fix uninitialized variable in `--force-if-includes` reflog walk
**[2026/09/04/21-01-20 by Tyler Cipriani]**

Tyler Cipriani posted a two-patch bugfix series targeting the `--force-if-includes` safety mechanism in `git push`. The series addresses a long-standing flaw where the feature incorrectly checks the reflog of the *local* branch whose name matches the *remote* destination branch (e.g., `main` when pushing to `origin/main`), rather than the branch actually being pushed (e.g., `src` when pushing `src:main`).

The first patch fixes the core logic in `remote.c` to consult the reflog of the pushed ref (`ref->peer_ref->name`) instead of the destination ref (`ref->name`). It adds special handling for `HEAD`-based pushes (resolving the symref) and detached HEAD states (rejecting the push by default). The second patch adjusts the advice message for detached HEAD scenarios, introducing a new `advice.forceIfIncludesDetachedHead` config knob and a `ref->unverifiable` flag to distinguish these rejections from remote-update rejections.

The series includes eight new test cases in `t/t5533-push-cas.sh` covering mismatched local/remote names, `HEAD`-based pushes, and detached HEAD states. The commit message cites two prior reports of false rejections (from 2023 and 2025) and explains how the same root cause could enable destructive pushes.

The implementation updates all relevant code paths (direct pushes, transport helpers, and send-pack) and maintains consistency with existing `--force-with-lease` behavior. The only potential point of discussion is the detached HEAD policy, but the author's rationale for rejecting such pushes by default is well-argued.

### git maintenance: improve rerere gc heuristic
**[2026/09/04/14-48-44 by Junio C Hamano]**

Junio C Hamano endorsed Patrick Steinhardt's v2 series improving the `git maintenance` heuristic for the `rerere gc` task. The series replaces the hard-coded 60-day cutoff with a dynamic heuristic that estimates stale `rerere` entries and triggers the task only when the estimated count meets or exceeds a configurable threshold (default: 512).

The v2 iteration addressed all prior feedback, including:
- Restoring the `prune_one()` helper function
- Clarifying configuration handling
- Renaming `rerere_gc_estimate()` to `rerere_gc_needed()`

Junio praised these changes as making the code "much easier to read" and provided a concise summary of the race condition the series addresses: lock contention between `git maintenance` and `git rebase` due to `MERGE_RR.lock` and `LOCK_DIE_ON_ERROR`. The series reduces the *frequency* of `rerere gc` runs but does not alter the locking mechanism itself.

The series received a tested review from Thomas Bachem, who confirmed it works as described and demonstrated the heuristic's correctness with both synthetic and real-world `rerere` entries. Derrick Stolee also reaffirmed his "LGTM" after verifying the addressed feedback.

### refs: unify terminology for "ref storage"
**[2026/09/04/10-36-01 by Patrick Steinhardt]**

Patrick Steinhardt posted an 11-patch series addressing long-standing inconsistencies in Git's terminology for the "ref storage format." Currently, the same concept is called `--ref-format=` on the command line, `refStorage` as a repository extension, and `GIT_REFERENCE_BACKEND` in environment variables. This inconsistency creates confusion, especially as the project prepares to introduce an analogous "object storage" extension.

The series systematically renames all occurrences of "ref format" to "ref storage":
- `--ref-format=` → `--ref-storage=` (with backward-compatible aliases)
- `GIT_REFERENCE_BACKEND` → `GIT_REF_STORAGE` (with backward-compatible aliases)
- `init.defaultRefFormat` → `init.defaultRefStorage` (with backward-compatible aliases)

The series also centralizes URI parsing for reference storage backends, enabling `--ref-storage=` to accept URIs like `files://path/to/repo` with payloads. This capability was already present in other parts of the codebase but not exposed here.

The motivation is clear: the current terminology is a "huge mess" that will only worsen with the upcoming object storage extension. The series touches built-in commands (`init`, `clone`, `refs`, `submodule`, `rev-parse`), documentation, test scripts, and shell completions across 39 files.

Junio C Hamano raised a design-level concern about the chosen term "ref-storage," questioning whether it clarifies the concept more effectively than "ref-format." Both terms are ambiguous about what aspect of refs they govern, and no alternative was proposed.

Karthik Nayak provided substantive reviews, identifying minor inconsistencies in help text placeholders and missing environment variable macros. The series is well-structured and largely uncontroversial, with the terminology question as the primary open issue.

### Regression reports resolved
**[2026/09/04/23-42-24 by Ramkumar Ramachandra]**

Ramkumar Ramachandra identified CrowdStrike Falcon's real-time file scanning as the root cause of three previously reported Git 2.55.0 regressions on macOS:
1. `git pull` failing with "dangling remote ref"
2. `git diff` exiting cleanly despite a dirty worktree
3. `git checkout -` failing with "invalid ref" after a pull

The symptoms were isolated to `llvm/llvm-project.git`, a massive repository with high update frequency. Ramkumar confirmed that CrowdStrike Falcon's exhaustive file scanning was racing against Git's high-frequency operations, producing random failures. The explanation accounts for the symptoms' non-reproducibility and repository-scale dependency.

This resolution closes the investigation with no action required from the Git project. The thread provides valuable context for future reports involving security software interference.

## In brief
- **[2026/09/04/18-47-35 by Erik Cervin Edin]** Erik Cervin Edin acknowledged Junio's observation about redundant option-checking logic in the `--fixup` message specification series and confirmed revisiting it after travel.
- **[2026/09/04/14-37-06 by K Jayatheerth]** K Jayatheerth outlined a two-phase plan to address architectural duplication in the `git repo info` path-keys series: merge v5 as-is, then follow up with a refactoring series to extract shared logic into a helper library.
- **[2026/09/04/10-51-33 by Siddharth Shrimali]** Siddharth Shrimali agreed with Samuel Bronson's proposal to change the index validation guard in `git repack --drop-filtered` from a fatal error to a warning that skips index-referenced blobs.
- **[2026/09/04/04-20-11 by Emmanuel Ugwu]** Emmanuel Ugwu asked whether now is a good time to tackle the deferred `#leftoverbits` item—standardizing ref-counting types to `size_t`—or if the project would prefer to wait until `hn/branch-delete-merged` has settled.
- **[2026/09/04/22-28-07 by Karthik Nayak]** Karthik Nayak confirmed the technical rationale for eliminating an implicit dependency on `the_repository` in the `cache-tree` subsystem in Patrick Steinhardt's ODB refactoring series.
- **[2026/09/04/07-44-36 by Thomas Bachem via GitGitGadget]** Thomas Bachem posted v2 of the locking patch in the `rerere` race-condition series, adding configurable locking timeouts and addressing all review feedback.
- **[2026/09/04/05-00-57 by Elijah Newren]** Elijah Newren proposed an additional test case for the heap-buffer-overflow bug in pathspec handling, using an exclude pathspec whose length exactly matches the common prefix of positive pathspecs.
- **[2026/09/04/12-44-33 by Aleksei Sviridkin]** Aleksei Sviridkin posted v2 of the `--force-if-includes` bugfix patch, removing the `Assisted-by: LLM` trailer and adjusting test cleanup placement.
- **[2026/09/04/07-53-44 by Thomas Bachem via GitGitGadget]** Thomas Bachem posted v2 of the series disabling auto-maintenance during sequencer operations, expanding scope to cover `git cherry-pick` and `git revert`.
- **[2026/09/04/20-35-51 by Mark C. Chu-Carroll via B4 Relay]** Mark C. Chu-Carroll posted a 2-patch series replacing `test -f` with `test_path_is_file` in 61 test scripts to improve test robustness.