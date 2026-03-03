# Git Mailing List Digest - 2025/06/11

**The day in brief.** A busy day with 74 emails across 25 threads, featuring significant progress on several fronts. Key highlights include the finalization of the promisor-remote protocol validation series, maintainer approval for a stash branch name fix, and ongoing discussions about submodule remote handling and batch branch operations. The day also saw Windows-specific CI fixes and performance-oriented refactoring of byte-swapping code.

## Notable threads

### Promisor-remote protocol validation finalized

Christian Couder's 5-patch series extending the promisor-remote protocol to support configurable validation of remote attributes has reached its final form (v4) after extensive review. The series allows servers to advertise additional fields like `partialCloneFilter` and `token` via `promisor.sendFields`, while clients can validate these using `promisor.checkFields`. Key changes in v4 include replacing `struct strvec` with `struct promisor_info` per Junio's suggestion, improved documentation formatting, and standardized terminology. The series has addressed all feedback from Patrick Steinhardt, Karthik Nayak, and Junio Hamano, with CI tests passing, indicating readiness for merging.

### Stash` branch name fix approved

K Jayatheerth's fix for `git stash list` branch display corruption in submodule contexts has been approved by Junio Hamano for merging. The patch addresses a subtle issue where stashing changes in a superproject would incorrectly display the submodule's branch name rather than the superproject's. The solution uses `xstrdup()` to preserve the correct branch name through submodule operations, with proper memory management. The only remaining issue is cosmetic trailing whitespace that doesn't affect functionality. This concludes a thorough review process that validated the technical approach and test coverage.

### Submodule remote lookup improvements

Jacob Keller's 6-part series refactoring Git's submodule remote lookup logic introduces URL-based remote matching as the primary lookup method before falling back to existing heuristics. The final patch adds `repo_remote_from_url()` to scan a repository's remotes for URL matches, modifying `repo_get_default_remote()` to use this when provided. This addresses cases where users rename their default remote, preventing submodule updates from failing due to looking for a non-existent "origin". The series includes preparatory infrastructure work like removing `the_repository` dependencies from remote.c and fixing a branch release crash, culminating in more robust submodule remote handling.

### `the_repository` removal progress

A refactoring patch moving `core.sparsecheckout` configuration from a global variable to the `repo_settings` struct sparked an interesting design discussion between Junio Hamano and Patrick Steinhardt. Junio questioned the use of getter/setter functions that internally call `prepare_repo_settings()`, suggesting simpler patterns used elsewhere in the codebase. The exchange clarified architectural principles for handling repository configuration variables, establishing that commonly-used settings like `core.sparsecheckout` should use direct access after initialization rather than getter/setter patterns. This guidance will influence future similar migrations in the ongoing `the_repository` removal effort.

### Batch branch operations debate

The discussion about implementing batch branch repointing functionality has evolved into a porcelain-vs-plumbing debate. Andrea Stacchiotti maintains preference for a `git branch` solution despite Junio Hamano's suggestion to use `update-ref --stdin`. Junio challenges both the technical justification and UI design, questioning the need for new flags when argument count could implicitly indicate batch mode. The thread remains unresolved, highlighting Git's ongoing tension between plumbing purity and user-facing convenience.

## In brief

**Git for Windows 2.50.0-rc2** -- Johannes Schindelin announces a platform-specific update with bundled dependency upgrades and fixes for ReFS drive operations and long branch name handling.

**Mailmap bugfix approved** -- A fix for incorrect mailmap application in `git cat-file` when processing commits with different author/committer identities has been approved after thorough review, addressing buffer handling issues in `ident.c`.

**Merge compact summary config** -- A patch extends `merge.stat` configuration to support the new `--compact-summary` format, allowing users to set their preferred diffstat format via `merge.stat=compact`.

**CI Windows Coverity fix** -- Johannes Schindelin submits a two-patch series fixing DLL conflicts in Windows CI builds by adjusting PATH ordering for Coverity tools and adding build log output on failure.

**Release notes polish** -- Kristoffer Haugsbakk submits several editorial improvements to the 2.50.0 release notes, standardizing terminology and fixing typos in function names.

**Byte-swapping refactoring** -- Sebastian Andrzej Siewior begins a 6-patch series addressing byte-order handling in Git's bswap.h, starting with reverting a problematic change and adding support for `__BYTE_ORDER__` macro.

## On the radar

**`git repo-info` development** -- Karthik Nayak continues mentoring Lucas Seiki Oshiro's GSoC project to extract repository metadata functionality from `git rev-parse` into a dedicated command, with recent focus on documentation and test improvements.

**Rebase trailer integration** -- The discussion about in-process trailer handling for `git rebase` is on hold pending Phillip Wood's return, with consensus that the functionality should be refactored into a shared helper first.