# Git mailing list daily digest for 2026/09/04

## The day in brief
The Git project saw significant progress on several fronts today. The `receive-report` hook series reached v7, addressing all prior feedback and preparing for integration. A major refactoring effort to unify "ref storage" terminology across Git commands was posted, touching 39 files. Several bugfixes advanced, including fixes for `--force-if-includes` and `rerere` race conditions. The day also brought clarity to a macOS regression caused by security software interference.

## Notable threads

### `receive-report` hook reaches v7
**[Thread root: 2026/08/18/07-55-55]**

Karthik Nayak posted v7 of the four-patch series introducing the `receive-report` hook for `git-receive-pack`. This hook allows server administrators to intercept and modify the status report sent to clients after ref updates are committed but before the response is finalized.

The series is now feature-complete and addresses all prior feedback. Key improvements in v7 include:
- Unified report generation by merging `report()` and `report_v2()` functions
- Code style improvements with a `switch` statement replacing an `if/else` chain
- Rebase onto `master` resolving conflicts with Justin Tobler's concurrent "pluggable writes" series

The hook is motivated by GitLab's need to implement multi-version concurrency control (MVCC), where the final status must reflect operations occurring after `ref_transaction_commit()`. The design intentionally decouples the reported status from the actual repository state, allowing a hook to report a push as failed while still committing ref updates. This trade-off is explicitly acknowledged in the commit message and justified by GitLab's use case.

The series is queued in `seen` and ready for `next`. All four patches received positive reviews, with no outstanding technical objections.

### Ref storage terminology unification
**[Thread root: 2026/09/04/10-36-01]**

Patrick Steinhardt posted an 11-patch series to unify Git's inconsistent terminology for "ref storage format" across CLI options, environment variables, config keys, and documentation. Currently, the same concept is called `--ref-format=` on the command line, `refStorage` as a repository extension, and `GIT_REFERENCE_BACKEND` in environment variables.

The series systematically renames all occurrences to "ref storage" for consistency:
- `--ref-format=` → `--ref-storage=` (with backward-compatible aliases)
- `GIT_REFERENCE_BACKEND` → `GIT_REF_STORAGE`
- `init.defaultRefFormat` → `init.defaultRefStorage`

The series also centralizes URI parsing for reference storage backends, enabling `--ref-storage=` to accept URIs like `files://path/to/repo`. This prepares for an analogous "object storage" extension.

Karthik Nayak provided substantive review, calling the series "relatively quite an easy read" despite its length. He identified two minor issues:
1. The help text placeholder for `--ref-storage=` still uses "format" instead of "ref-storage"
2. A missing `GIT_DEFAULT_REF_STORAGE_ENVIRONMENT` macro

Junio C Hamano raised a design-level concern about whether "ref-storage" is meaningfully clearer than "ref-format," noting both terms are ambiguous about what aspect of refs they govern.

### `--force-if-includes` bugfix
**[Thread root: 2026/09/04/21-01-20]**

Tyler Cipriani posted a two-patch series fixing a long-standing flaw in the `--force-if-includes` safety mechanism. The feature currently checks the reflog of the *local* branch whose name matches the *remote* destination branch, rather than the branch actually being pushed. This can lead to false rejections or unintended data loss.

The first patch fixes the core logic to consult the reflog of the pushed ref. The second patch adjusts advice messages for detached HEAD scenarios, introducing a new `advice.forceIfIncludesDetachedHead` config knob.

The series includes eight new test cases covering mismatched local/remote names, `HEAD`-based pushes, and detached HEAD states. The author cites two prior reports of false rejections (2023, 2025) with the same root cause.

### `rerere` race condition fixes
**[Thread root: 2026/09/02/08-31-37]**

Thomas Bachem posted v3 of a patch addressing a race condition between `git rebase` and background `rerere gc` maintenance. The patch adds configurable locking timeouts to `rerere_setup()`, allowing foreground operations to wait for the lock rather than failing immediately.

Key improvements in v3:
- Configurable timeout (`rerere.lockTimeout`, default 1000ms)
- `BUG()` assertions for incompatible flag combinations
- Clarified documentation about command behavior under lock contention

The patch addresses all prior feedback from Junio C Hamano, Phillip Wood, and Patrick Steinhardt. Junio clarified that after waiting, commands should fail rather than proceed, as a held lock signals another process is actively modifying the `rr-cache`.

### macOS regression resolved
**[Thread root: 2026/09/04/18-05-43]**

Ramkumar Ramachandra reported three regressions in Git 2.55.0 on macOS:
1. `git pull` failing with "dangling remote ref"
2. `git diff` exiting cleanly despite a dirty worktree
3. `git checkout -` failing after a pull

The issues were isolated to `llvm/llvm-project.git`, a massive repository with high update frequency. After investigation, Ramkumar identified CrowdStrike Falcon's real-time file scanning as the root cause, with the security software racing against Git's high-frequency operations.

## In brief

- **[2026/05/18/11-22-24]** Erik Cervin Edin acknowledged redundant option-checking logic in the `--fixup` message specification series and plans to address it after travel.
- **[2026/07/16/01-21-31]** K Jayatheerth outlined a two-phase plan to address architectural duplication in the `git repo info` path-keys series: merge v5 as-is, then follow up with a refactoring series.
- **[2026/07/16/13-28-41]** Siddharth Shrimali agreed to change the index validation guard in `git repack --drop-filtered` from a fatal error to a warning that skips index-referenced blobs.
- **[2026/07/24/09-11-53]** Emmanuel Ugwu asked about timing for standardizing ref-counting types to `size_t`, with the consensus being to wait until `hn/branch-delete-merged` settles.
- **[2026/08/25/20-46-42]** Phillip Wood identified output format ambiguity in `-z` mode for the `git var` extension and proposed adopting `git config -z`'s format.
- **[2026/09/01/11-08-59]** Karthik Nayak provided substantive reviews of Patrick Steinhardt's ODB refactoring series, confirming the technical rationale for eliminating implicit dependencies.
- **[2026/09/02/13-04-09]** Elijah Newren proposed additional test cases for the pathspec negative prefix fix to make the heap-buffer-overflow bug deterministic without AddressSanitizer.
- **[2026/09/03/01-25-48]** Aleksei Sviridkin posted v2 of a patch fixing an uninitialized variable in `--force-if-includes`, removing the `Assisted-by: LLM` trailer after Junio's policy clarification.
- **[2026/09/03/09-04-56]** Phillip Wood and Junio C Hamano debated test coverage for the `CHERRY_PICK_HEAD` documentation patch, with Phillip arguing for minimal coverage and Junio emphasizing project policy.
- **[2026/09/03/12-55-23]** Thomas Bachem posted v2 of a series deferring auto-maintenance during sequencer operations, expanding scope to cover `git cherry-pick` and `git revert`.
- **[2026/09/04/20-35-51]** Mark C. Chu-Carroll posted a two-patch series replacing `test -f` with `test_path_is_file` in 61 test scripts for better error messages.