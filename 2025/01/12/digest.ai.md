# Git Mailing List Digest — 2025/01/12

## The day in brief

A quiet Sunday with 7 emails across 5 threads, mostly focused on bugfixes and documentation. The most notable developments include a fix for HEAD handling in bare repositories with mixed mirror/non-mirror remotes and a discussion about reclassifying `git-cherry` as a porcelain command.

## Notable threads

### Fix for HEAD handling in bare repositories with mixed remotes

A regression introduced in Git 2.48.0 caused bare repositories with multiple remotes to incorrectly overwrite the HEAD symref when fetching from non-mirror remotes. The bug, introduced in commit b1b713f722, made bare repositories treat all remotes as mirrors, causing HEAD to point to non-existent branches. A proposed fix from today modifies the `set_head` logic in `builtin/fetch.c` to distinguish between mirror and non-mirror remotes in bare repos, only overwriting HEAD for true mirrors while properly setting remote-specific HEAD symrefs otherwise. The patch includes two new test cases verifying the corrected behavior and maintains existing functionality for mirror remotes.

### `git-cherry` classification debate

A documentation patch proposes reverting `git-cherry` from plumbing back to porcelain classification (specifically under "ancillary interrogators"). The argument centers on whether command frequency of use should determine its status - while `git-cherry` is rarely used, its primary value is as a user-facing tool for checking which commits have or haven't been applied to another branch. The patch also fixes a regression in Bash completion that occurred when the command was moved to plumbing. The discussion highlights ongoing questions about where to draw the line between porcelain and plumbing commands, particularly for stable-output commands that serve user workflows.

## In brief

A refactoring patch fixes memory leaks and inconsistent behavior in ref-filter code when handling ahead-behind and is-base calculations, affecting several commands (`branch`, `for-each-ref`, `tag`, `verify-tag`).

Bence Ferdinandy confirmed that a reported issue with tag fetching in shallow clones (`--depth=1`) was already fixed in commit 6c915c3f85 ("fetch: do not ask for HEAD unnecessarily", 2024-12-06).

The automatic setting of remote HEAD references functionality (requested by Caleb Cushing) has been implemented in a new release with configuration options documented at git-scm.com under `remote.<name>.followRemoteHEAD`.