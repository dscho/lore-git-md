Here's the daily digest for April 20, 2026:

### The day in brief
A busy with Git v2.54.0's release, ongoing refactoring efforts, and several technical discussions. Key highlights include the completion of Patrick Steinhardt's `the_repository` removal in setup.c and resolution of a commit signing regression for non-UTF-8 messages.

### Notable threads

**Commit signing regression fixed**  
brian m. carlson addressed a regression where commit signatures became invalid for non-UTF-8 messages since Git 2.45.0. The two-patch series first renames `verify_utf8()` to `ensure_utf8()` to better reflect its behavior, then fixes the core issue by moving UTF-8 processing before signing operations. This restores the pre-2.45.0 behavior where signatures remain valid even with non-UTF-8 commit messages.

**`the_repository` removal in setup.c completed**  
Patrick Steinhardt's 18-patch series to eliminate `the_repository` usage from the setup subsystem reached completion after thorough review. The changes convert core functions like `setup_git_directory()` and `setup_work_tree()` to use explicit repository parameters. Elijah Newren provided the final review, noting one minor optimization opportunity in patch 16. Junio Hamano raised questions about performance implications of removing the `initialized` flag but overall approved the approach.

**Ref backend consolidation**  
Karthik Nayak's 8-patch series to standardize object validation and peeling across Git's reference backends (files, packed, reftable) was completed. The changes move common logic into the shared refs layer, eliminating backend-specific implementations. The series demonstrates the project's ongoing effort to reduce code duplication and ensure consistent behavior across storage backends.

**Git v2.54.0 released**  
Junio Hamano announced Git v2.54.0, featuring 771 non-merge commits from 137 contributors. Notable changes include the experimental `git history` command, enhanced `git replay`, improved worktree documentation, and continued progress on `the_repository` removal and ODB abstraction. The Git for Windows release followed shortly after, dropping `git svn` due to maintenance burdens.

### In brief

**`--word-diff` for commit -v** -- A new patch adds `--word-diff` support to `git commit -v`, enabling word-level diffs in the commit template for structured data formats.

**Merge-ORT edge case fix** -- Elijah Newren fixed an interaction between cached renames and trivial resolutions in the merge-ort engine, adding a test case to prevent regression.

**Test suite robustness** -- Patrick Steinhardt's series to make Git's test suite compatible with `set -e` (errexit) received final review from Junio, with only minor documentation consistency questions remaining.

**Ref verification lock files** -- Karthik Nayak fixed inconsistent handling of lock files during ref verification, with Junio suggesting the check might better belong in `files_fsck_ref()` itself.

**Rebase status display** -- Phillip Wood improved `git status` display during interactive rebase, with Tian Yuchen identifying an edge case in the abbreviation logic for short refnames.

### On the radar

**Source tree reorganization** -- Patrick Steinhardt's RFC to move libgit.a components into a "lib/" directory has reached design consensus, with discussion now focusing on transition timing and downstream impact.

**Linux fsmonitor** -- Paul Tarjan's Linux fsmonitor implementation appears ready but lacks in-depth reviews beyond Patrick Steinhardt's, potentially delaying integration.