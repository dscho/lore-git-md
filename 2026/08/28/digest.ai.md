# Git mailing list daily digest for 2026/08/28

## The day in brief
The Git mailing list saw significant progress on several fronts today. Elijah Newren’s bugfix series for unsafe commit operations during conflict resolution reached v3, addressing all substantive feedback and removing a controversial restriction. Junio C Hamano proposed a sweeping architectural refactoring of `git checkout`, `switch`, and `restore` to eliminate the monolithic `checkout_main()` helper. A security feature adding OCSP staple validation (`http.sslVerifyStatus`) awaits a final decision on test scope, while a worktree repair bugfix was marked ready for `next`. The `--[no-]range-diff-notes` feature was abandoned due to usability concerns.

## Notable threads

### OCSP staple validation (`http.sslVerifyStatus`) awaits test-scope decision
The `http.sslVerifyStatus` security feature, which enables OCSP staple validation for HTTPS connections, is technically complete and ready for `next`. All prior review feedback—including Junio C Hamano’s documentation wording tweak in v6—has been addressed. The only remaining question is whether to merge Patrick Steinhardt’s additional OCSP test harness (`t5585-http-ssl-ocsp.sh`), which exercises positive cases (valid staples) but adds complexity.

Today, the author (Grayson Gordon) asked Junio for guidance on this trade-off between thoroughness and succinctness. Junio responded by asking Patrick which parts of the test harness feel uncertain or overly complex, noting the tests appear to cover essential scenarios but may not explicitly verify the case where the server does not provide a staple. The thread is now awaiting Patrick’s input to resolve this final question before the patch graduates to `next`.

### Worktree repair bugfix marked ready for `next`
Yoichi NAKAYAMA posted v4 of a bugfix patch targeting worktree repair functionality. The patch corrects how Git detects and handles relative versus absolute paths in `.git` files when `worktree.useRelativePaths` is toggled. The existing code failed to detect mismatches because `read_gitfile_gently()` always returns an absolute path, obscuring the original path style.

The v4 iteration expands the commit message to justify why `read_gitfile_raw()` omits repository validation (`is_git_directory()`), addressing Junio’s feedback that the v3 message was "too sketchy." The new function returns the path verbatim (absolute or relative) without normalization, enabling the worktree repair logic to detect mismatches between the configured path style and the `.git` file contents. Junio marked the patch ready for `next`, signaling it is ready for integration testing.

### `--[no-]range-diff-notes` feature abandoned due to usability concerns
Kristoffer Haugsbakk abandoned the `[PATCH 0/3] format-patch: add --[no-]range-diff-notes` series in response to Junio C Hamano’s substantive design critique. The feature aimed to allow independent control over whether Git notes are included in the range-diff section versus the patches themselves.

The core issue was usability: the current implementation does not support the intuitive use case where `--no-notes --range-diff-notes` suppresses notes in the patches but *includes* them in the range-diff. Junio explicitly stated that the interaction between the new option and the existing `--notes` cannot be defined in simple terms, making the feature’s semantics conceptually flawed. Kristoffer acknowledged the problem and chose to drop the topic entirely rather than propose a redesign. The series touched `builtin/log.c`, `log-tree.c`, `revision.c`, `revision.h`, and tests in `t/t3206-range-diff.sh`.

### ODB alternates refactoring raises security and locking questions
Junio C Hamano raised a security and correctness concern about the sixth patch in Patrick Steinhardt’s 8-patch series refactoring ODB alternates handling. The patch writes the `info/alternates` file in append mode (`"a"`) rather than using Git’s standard lockfile API, which could lead to partial writes or TOCTOU (time-of-check-to-time-of-use) races during repository creation.

Junio noted that while the operation happens only once during repository creation, using the lockfile API would be the safer and more conventional choice. Toon Claes also questioned why the new implementation (introduced in patch 6) does not use a lockfile, directly challenging the correctness of the refactoring. The series aims to remove the ability to write alternates after repository creation, simplifying the ODB interface, but these questions may delay its progression until the locking strategy is resolved.

### Bugfix series for unsafe commit operations during conflict resolution reaches v3
Elijah Newren posted v3 of a five-patch bugfix series that extends Git’s protection against unsafe commit operations during conflict resolution. The series blocks both `git commit --amend` and `git commit <paths>` during conflict resolution for `git am`, `git revert`, and all forms of `git rebase`, except for interactive rebase directives like `squash` or `edit`.

The v3 iteration addresses all substantive feedback from earlier reviews:
- Renamed `FROM_REBASE_PICK` to `FROM_REBASE_EMPTY` and `is_from_rebase()` to `is_from_rebase_empty()` to clarify these symbols relate to commits that became empty during a rebase.
- Removed the restriction on `git commit <paths>` when a rebase pick becomes empty, addressing Junio’s concern that the result would still be an empty commit and thus harmless.
- Reworded error messages to better distinguish between conflict stops and clean interactive rebase stops.

The series is now technically sound and ready for final review. Junio approved the changes in patches 2-5, and the only remaining feedback is a stylistic suggestion about the naming of the new enum values and helper functions. The series touches `builtin/commit.c`, `sequencer.c`, `sequencer.h`, and test coverage in `t3404-rebase-interactive.sh`, `t3507-cherry-pick-conflict.sh`, and `t4151-am-abort.sh`.

### Architectural refactoring of `git checkout`, `switch`, and `restore`
Junio C Hamano posted an 8-patch series refactoring the internals of `git checkout`, `git switch`, and `git restore` to eliminate the monolithic `checkout_main()` helper. The series is part of the broader libification effort, where reusable logic is factored out of built-in commands into standalone helpers that accept a `struct repository *` parameter.

The series begins with isolated cleanups (patches 1-3) that could stand alone, such as passing `cb_option` explicitly to branch name parsers and moving branch-name validation into `checkout_branch()`. Patches 4-5 extract utility functions for option validation, pathspec handling, branch setup, and tracking. Patch 6 is the core refactoring: it rewrites the three top-level command entry points (`cmd_checkout()`, `cmd_switch()`, `cmd_restore()`) to call the extracted helpers directly, bypassing `checkout_main()`. Patches 7-8 are cleanup and illustration steps, including moving `post_checkout_hook()` to a top-level file to demonstrate libification.

The series is explicitly marked as an early draft for discussion, not for merging to `seen`, due to potential conflicts with in-flight topics. Junio’s self-review of the final patch identified a shortcoming: `post_checkout_hook()` still hardcodes `the_repository` and `the_hash_algo`, which undermines its reusability in non-global contexts. The series aligns with the recent architectural direction of redirecting the `the_repository` removal effort toward utility code outside `builtin/`.

## In brief
- **[PATCH] builtin/*: stop passing the_repository to is_bare_repository()**: Junio C Hamano clarified that the `bool has_repo` design for built-in commands is motivated by safety and clarity, not efficiency, and redirected the libification effort toward factoring reusable logic out of built-in commands into standalone helpers. Hardik Kumar and D. Ben Knoble acknowledged the architectural direction.
- **[PATCH] Fix typos**: Hardik Kumar posted a contributor housekeeping patch fixing two typos: "fractionnal" → "fractional" in `versioncmp.c` and "similiarity" → "similarity" in `t/t0022-crlf-rename.sh`.
- **[PATCH] dir: do not strip prefix from negative pathspecs**: Diogo Castro posted a bugfix patch targeting negative pathspec handling in `git ls-files` and `git add`. The patch skips prefix-stripping for negative pathspecs to prevent them from degenerating into empty strings or reading out of bounds. Junio asked whether the common prefix should be calculated across both positive and negative pathspecs, rather than skipping prefix-stripping for negative pathspecs entirely.
- **What’s cooking in git.git (Aug 2026, #12)**: Junio’s status report highlighted 567 non-merge commits in `master` since Git 2.55, with 76 topics in `next` and 215 in `seen`. Two topics graduated to `master`: a documentation fix for `git format-rev` and a heap-use-after-free fix in object name parsing. Notable stalled topics include `ec/commit-fixup-options` and `ps/libgit-in-subdir`, while cooking topics include `kn/reftable-optimize-reloading` and `en/no-amend-during-conflicts`.