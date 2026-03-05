# Git Mailing List Digest — 2025/09/13

## The day in brief

A moderately busy Saturday with 28 emails across 11 threads, featuring progress on several technical fronts. The standout developments include final refinements to the command deprecation framework's alias handling, a comprehensive solution for case-insensitive filesystem issues in batched reference updates, and new worktree independence features for rebase operations. Meanwhile, troubleshooting threads around mergetool variables and `safe.directory` behavior revealed deeper implementation complexities.

## Notable threads

### Alias recursion handling finalized for command deprecation

Jeff King and Kristoffer Haugsbakk polished the alias recursion detection in the command deprecation framework, moving tracking logic into `handle_alias()` as suggested in earlier reviews. The v4 changes improve error reporting for recursive aliases with clearer loop indication markers (==>), while maintaining all existing functionality. This completes the architectural work needed for alias shadowing of deprecated commands, with the series already maintainer-approved. The final discussion focused on edge cases in error message formatting rather than core functionality.

### Case-insensitive ref update conflicts resolved

A 4-patch series from Gabriel Scherer reached its final iteration, comprehensively addressing batched reference update issues on case-insensitive filesystems. The solution now properly handles four conflict types: direct case collisions, file/directory conflicts, directory/file conflicts, and pre-existing lock file conflicts — each with distinct error categorization (`REF_TRANSACTION_ERROR_CASE_CONFLICT`, `REF_TRANSACTION_ERROR_CREATE_EXISTS`). The implementation allows non-conflicting updates to proceed while failing only the problematic references, striking a balance between pre-2.51's silent overwrites and post-2.51's total batch failures. Junio Hamano's review feedback has been incorporated, with the reftable backend noted as the long-term solution for these filesystem limitations.

### Worktree independence extended to rebase

New contributor Gabriel Scherer proposed a series adding `--ignore-other-worktrees` support to `git rebase`, mirroring the existing checkout option. The changes include improved advice messages when operations fail due to branch conflicts across worktrees, suggesting `--detach` or `--ignore-other-worktrees` as workarounds. The implementation refactors `die_if_checked_out()` to support messaging and adds thorough test coverage in t/t3400-rebase.sh. Review feedback from Ben Knoble and Kristoffer Haugsbakk focused on test completeness and advice wording, with the core functionality appearing sound. The series lays groundwork for potential future `branch.ignoreOtherWorktrees` configuration.

### Mergetool variable passing puzzles persist

Randall Becker's investigation into mergetool variable passing reached an impasse as Johannes Sixt declined to further analyze what he called the "far too convoluted" implementation. The thread clarified that BASE/LOCAL/REMOTE/MERGED variables are shell variables rather than environment variables, explaining why they don't appear in `env` output. The discussion revealed a documentation mismatch for custom mergetools, which can't access these variables through the same mechanism as built-in tools. With no clear path forward, the thread may require maintainer input to determine whether the current behavior is intentional or needs adjustment.

## In brief

**gitk remote branch display fix** — Johannes Schindelin corrected an edge case where deleted remote tracking branches caused gitk to crash when opening the "Tags and Heads" view, adding proper existence checks for upstream references.

**gitk window geometry persistence** — A patch makes the "Tags and Heads" window remember its size and position between sessions, storing the geometry in gitk's config file and handling multi-monitor edge cases.

**Outreachy mentorship offer** — Usman Akinyemi volunteered to co-mentor for Git's December 2025 Outreachy cohort, though sponsorship challenges for the program remain unresolved.

**`git apply` fuzzy match limitations** — Guo Tingsheng provided a concrete JavaDoc example showing how structurally similar method documentation can cause patch misapplication, confirming this as an expected (if unfortunate) limitation of line-based matching.

## On the radar

**`safe.directory` server-side confusion** — Carlo Arenas Belón clarified that a reported `safe.directory` failure originates server-side, shifting troubleshooting focus but leaving open why a properly configured shared repository triggers the security check.