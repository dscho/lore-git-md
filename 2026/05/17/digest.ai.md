# Git Mailing List Digest - 2026/05/17 (Sunday)

**The day in brief.** A moderately active Sunday with 22 emails across 10 threads, featuring continued refinement of several technical series. Key developments include final design decisions pending for `git log --graph` visualization improvements, performance optimization approvals, and documentation standardization work. Two separate segfault reports in `git rebase --continue` during signing operations warrant attention from maintainers.

## Notable threads

### `git log --graph` root commit visualization approaches

The long-running discussion about improving `git log --graph` visualization for commits with excluded parents has reached its final design phase. Chandra Pratap's analysis today highlights the tradeoffs between two approaches for root commit separation: indentation versus blank lines. The technical equivalency is confirmed, leaving the decision as a pure UX choice between visual distinction (blank lines) and implementation simplicity (indentation). With all technical considerations now surfaced, this thread awaits Junio's final decision on which approach to adopt.

### `limit_list()` performance optimization approved

Junio C Hamano approved Kristofer Karlsson's priority queue optimization for `limit_list()`, marking it ready for merging after standard integration testing. Benchmarks show 1.6-4.3x speedups in merge-heavy repositories, with René Scharfe's additional testing confirming meaningful improvements even in linear histories. While Kristofer identified potential 3.3x speedups in `get_revision_1()`, Junio wisely deferred that as future work to keep the current change focused. This concludes a well-reviewed optimization that improves performance for commands using symmetric difference operations like `--left-right` and `--cherry-mark`.

### Merge commit support for `git history`

Johannes Schindelin's RFC series adding merge commit support to `git history` saw substantive technical discussion today. Phillip Wood identified an edge case where new conflicts could be silently introduced when only one merge parent is rebased. Dscho acknowledged this as a "really good idea" and is reworking the patches to incorporate Wood's suggested single-parent-rebase optimization alongside the original three-way merge composition approach. The thread also resolved test infrastructure questions, with Toon Claes' DSL syntax improvements being accepted. This series continues to evolve through careful technical collaboration.

## In brief

**Config file locking timeout follow-up** -- Johannes Schindelin shared implementation details from Microsoft's fork showing configurable timeouts working in practice, though the core maintainers had previously rejected this approach due to fundamental race condition concerns.

**Files ref backend fsck fix** -- Karthik Nayak's v3 patch to properly skip lock files during ref verification is now ready, having addressed all technical feedback from Junio, Christian Couder, and Patrick Steinhardt.

**Documentation standardization batch** -- Jean-Noël Avila sent a 5-patch series converting git-bisect, git-grep, git-am, git-apply, and git-imap-send man pages to the new AsciiDoc synopsis style, continuing the mechanical documentation standardization effort.

**`the_repository` removal thread-safety** -- Tian Yuchen and Junio confirmed a thread-safety issue in Patrick Steinhardt's setup.c conversion regarding static buffer usage in `is_inside_worktree()`, requiring resolution before the series can proceed.

## On the radar

**Rebase segfault reports** -- Two separate threads reported segfaults in `git rebase --continue` during commit signing operations, both suggesting NULL-dereferences in merge-ort. The more detailed report includes a backtrace from Git 2.54.0, while the other lacks technical specifics. These warrant investigation as potential serious bugs affecting signed rebase operations.