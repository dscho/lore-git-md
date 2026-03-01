# Git Mailing List Digest — 2025/01/12

## The day in brief

A quiet Sunday with just seven emails across five threads, mostly focused on bugfixes and documentation clarifications. The most notable developments include a fix for HEAD handling in bare repositories with mixed mirror/non-mirror remotes and a discussion about reclassifying `git-cherry` between porcelain and plumbing commands.

## Notable threads

### Fix for HEAD handling in bare repositories with mixed remotes

A regression introduced in Git 2.48.0 caused bare repositories with multiple remotes to incorrectly overwrite the HEAD symref when fetching from non-mirror remotes. The bug, introduced by commit b1b713f722, made bare repositories treat all remotes as mirrors, leading HEAD to point to non-existent branches after fetching from a second remote. The proposed fix modifies the `set_head` logic in `builtin/fetch.c` to properly distinguish between mirror and non-mirror remotes in bare repositories. The patch includes new test cases verifying that non-mirror fetches no longer interfere with existing HEAD in bare repos while maintaining proper remote HEAD handling. This appears to be a well-targeted solution for the specific regression, with clear test coverage for both the broken behavior and the corrected case.

### Documentation debate: Should git-cherry be porcelain or plumbing?

A documentation patch proposes reverting `git-cherry`'s classification from plumbing back to porcelain (specifically under "ancillary interrogators"). The argument centers on whether command frequency of use should determine its status, with the author contending that despite being rarely used, `git-cherry` serves a high-level workflow purpose (checking which commits have or haven't been applied to another branch) rather than being primarily a script-building block. The patch also fixes a regression in Bash completion that occurred when the command was moved to plumbing. The discussion highlights ongoing questions about how to classify commands that have stable output (a plumbing characteristic) but serve user-facing workflows (a porcelain characteristic).

## In brief

The remote HEAD reference behavior thread saw confirmation that the requested automatic setting functionality has been implemented in a new release, with configuration options documented at git-scm.com under `remote.<name>.followRemoteHEAD`. A refactoring patch fixed memory leaks and inconsistent behavior in ref-filter code when handling ahead-behind and is-base calculations, affecting several commands that use ref-filter functionality. In the shallow clones thread, Bence Ferdinandy pointed out that a reported `git fetch` tag behavior issue in Git 2.48.rc0 was already fixed in commit 6c915c3f85, suggesting the regression should be resolved in the final 2.48 release.