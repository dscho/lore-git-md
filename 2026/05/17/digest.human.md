# Git Mailing List Digest - 2026/05/17 (Sunday)

**The day in brief.** A moderately active Sunday with 22 emails across 10 threads, featuring continued refinement of several technical series and two new bug reports. The most notable developments include final design decisions needed for Pablo Sabater's `git log --graph` improvements and Johannes Schindelin's progress on merge support for `git history`. Two separate segfault reports in `git rebase --continue` during signing operations warrant attention from the rebase and merge-ort maintainers.

## Notable threads

### `git log --graph` root commit visualization approaches

The long-running discussion about improving `git log --graph` rendering of commits with excluded parents has reached its final decision point. Chandra Pratap's latest analysis compares two approaches for root commit visualization: indentation (technically simpler) versus blank line separation (potentially clearer visually but requiring more invasive changes). With all technical tradeoffs now clearly laid out and no unresolved implementation concerns, the thread awaits Junio's decision on which visual style to adopt. Both solutions satisfy the core requirement of distinguishing root commits from their children, making this purely a UX choice.

### `git history` merge support RFC progresses

Johannes Schindelin's RFC series adding merge commit support to `git history` saw substantive technical discussion today. The thread resolved test infrastructure syntax questions (adopting Toon Claes' suggested DSL improvements) while grappling with Phillip Wood's identified edge case about silent conflict introduction when only one merge parent is rebased. Dscho acknowledged Wood's alternative single-parent-rebase optimization as "a really good idea" that may be incorporated, though implementation details remain to be worked out. The series continues to evolve through this collaborative review process.

### Files ref backend lock file handling finalized

Karthik Nayak's bugfix series addressing lock file processing in the files ref backend's consistency checks reached its final form (v3) today. The patch moves lock file skipping logic to the directory iteration level, ensuring all validation checks consistently ignore `.lock` files. This architectural improvement prevents lock files from being incorrectly processed by any current or future validation functions. With comprehensive test coverage and all prior technical concerns addressed, the solution appears ready for merging.

### `limit_list()` optimization approved

Junio C Hamano approved Kristofer Karlsson's performance optimization series for `limit_list()`, which replaces a linked list with a priority queue to achieve 1.6-4.3x speedups in merge-heavy repositories. The thread saw additional benchmark data from René Scharfe showing consistent 1.16-1.28x improvements even in Git's relatively linear history. While Kristofer identified potential optimizations in `get_revision_1()` (showing 3.3x speedups in monorepos), Junio confirmed these should be pursued separately. The approved changes will now proceed through standard integration.

## In brief

**Documentation standardization batch** -- Jean-Noël Avila sent a series converting five command manuals (git-bisect, git-grep, git-am, git-apply, git-imap-send) to the new AsciiDoc synopsis style, completing another batch in the ongoing documentation standardization effort.

**Config file locking timeout follow-up** -- Johannes Schindelin shared implementation details from Microsoft's fork about configurable timeouts for config file locking, validating the technical discussion in Jörg Thalheim's abandoned patch series.

**`the_repository` removal thread-safety concern** -- Tian Yuchen and Junio C Hamano identified a thread-safety issue with a static buffer in Patrick Steinhardt's `is_inside_worktree()` conversion, requiring resolution before the setup subsystem changes can proceed.

**Rebase segfault reports** -- Two separate threads reported segfaults in `git rebase --continue` during commit signing operations, with one providing a backtrace showing a NULL-dereference in merge-ort's tree parsing code.

## On the radar

**`the_repository` removal** -- Patrick Steinhardt's setup subsystem conversion now faces a confirmed thread-safety issue that must be addressed before integration, potentially requiring changes to the `is_inside_worktree()` caching mechanism.