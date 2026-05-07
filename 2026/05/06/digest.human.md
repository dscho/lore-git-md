# Git Mailing List Digest - 2026/05/06 (Wednesday)

**The day in brief.** A moderately busy day with 32 emails across 15 threads, featuring significant progress on several fronts. The standout developments include Johannes Schindelin's RFC series for merge support in `git history` and the finalization of the `--max-count-oldest` feature. Meanwhile, the git-gui bugfix series reaches v6 with comprehensive fixes for bare repository handling.

## Notable threads

### **Merge support for git history proposed**

Johannes Schindelin submitted an RFC series (5 patches) adding merge commit support to `git history`, the new history-rewriting command introduced in Git 2.54. The implementation handles 2-parent merges using a three-way merge composition approach (R=original parents' auto-merge, O=original merge, N=rewritten parents' auto-merge) while explicitly deferring octopus merges and merge reverts. The series includes a new `test-tool historian` for creating complex merge test scenarios and extensive test coverage. This represents a significant expansion of `git history`'s capabilities, though marked RFC to gather feedback on the approach before finalizing.

### **git-gui bare repository fixes reach v6**

Shroom Moo's bugfix series for git-gui's handling of bare repositories and missing worktrees reached its sixth iteration, now addressing all identified edge cases. The patches restructure the startup sequence to properly detect repository state before Tcl initialization, disable visualization features when no worktree is available, and fix environment variable handling to match `git -C` behavior. The series has incorporated extensive feedback from Johannes Sixt and Mark Levedahl, particularly around the tricky interaction between `GIT_DIR` and `GIT_WORK_TREE`. With consensus achieved on the technical approach, this appears ready for merging.

### **Final polish for --max-count-oldest feature**

The `--max-count-oldest` feature (showing oldest N commits in revision walks) received its final review round from Johannes Sixt, who suggested improvements to documentation clarity, error message formatting, and implementation location. Mirko Faina responded with plans to address all points in v7, including moving the counting logic into `get_revision_internal()` to fix `--boundary` interaction. This marks the feature as implementation-complete after seven iterations, with all substantive review feedback addressed and only final polish remaining before potential merging.

### **Ref backend consolidation nears completion**

Karthik Nayak's ref backend refactoring series (consolidating object validation and peeling logic) received confirmation from Toon Claes that the v4 iteration is in good shape. The series standardizes these operations across Git's reference backends (files, packed, reftable) by moving them into the shared refs layer. With Patrick Steinhardt having already approved the architecture and Toon confirming the error handling fixes, this long-running consolidation effort appears ready for merging once the minor stylistic refinements are applied.

## In brief

**Windows large-objects CI fix** -- Johannes Schindelin pointed to Patrick Steinhardt's diagnosis of a CI build failure in the Windows large-objects series, caused by glibc's C11 `_Generic` usage conflicting with clang's C99 checking on Ubuntu 26.04 runners.

**fetch --deepen regression fix** -- Samo Pogačnik submitted the final patch fixing a regression where `fetch --deepen` could incorrectly convert full clones to shallow ones, now properly making the operation a no-op for non-shallow repositories.

**maintenance prefetch error handling** -- Phillip Wood identified the root cause of intra-repository prefetch failures in `git maintenance` as coming from `connect.c:die_initial_contact()`, suggesting `remote.<name>.skipFetchAll` as a temporary workaround.

**RHEL 6 build failure** -- The discussion continued about Git 2.54's build failure on RHEL 6 due to OpenSSL conflicts, with D. Ben Knoble suggesting the reporter use `git bisect` to identify the problematic change themselves given the platform's unsupported status.

**Terminal output corruption** -- The thread about ANSI escape sequence handling saw further technical discussion, with René Scharfe clarifying the clear sequence's original purpose (progress line cleanup) while acknowledging Mikael Magnusson's point about background color persistence during line wrapping.

**reflog expire --dry-run output** -- Sven Weiland reported a minor inconsistency where `git reflog expire --dry-run` omits the standard "would" prefix in its output messages.

**rebase --update-refs edge case** -- A bugfix was proposed for `git rebase --update-refs` to prevent generating invalid instructions for non-branch refs when using `rebase.instructionFormat` with `%d`.

**git diff --word-diff whitespace** -- Vincent Lefevre reported that `git diff --word-diff` incorrectly highlights entire lines when only leading/trailing whitespace differs.

**Documentation typos** -- Andrew Kreimer submitted a mechanical patch fixing various spelling errors across documentation, comments, and test files using the `codespell` tool.

**git add -p word-diff** -- A feature enhancement was proposed adding a 'w' command to `git add -p` that displays the current hunk using word-diff formatting, particularly useful for reviewing changes in structured text files.