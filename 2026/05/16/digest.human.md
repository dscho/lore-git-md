# Git Mailing List Digest - 2026/05/16 (Saturday)

**The day in brief.** A moderately active Saturday with 42 emails across 11 threads, featuring substantive technical discussions on date parsing improvements, git-gui bugfixes, and performance optimizations. The most notable developments include Tuomas Ahola's v4 series fixing edge cases in Git's date approximation logic and Kristofer Karlsson's performance optimization for commit-reach.c now ready for merging.

## Notable threads

### Date approximation edge case fixes reach v4

Tuomas Ahola's four-patch series addressing edge cases in Git's date approximation logic (`approxidate`) has reached v4 with significant refinements. The series fixes longstanding issues in how special date/time strings like "noon" and "tea" interact with explicit day specifications. The key change in v4 implements Junio's preferred semantic where "today" means midnight rather than the current time, addressing a quirk where `git log --since=today` would often show no commits (since it compared against future timestamps). The implementation introduces a deferred adjustment mechanism using negative `tm_mday` values to handle combinations like "noon yesterday" correctly. With all review feedback addressed and comprehensive test coverage added, this series appears ready for final review and merging.

### git-gui bugfix series undergoes maintainer review

The extensive git-gui bugfix series addressing repository and worktree detection issues received thorough review from maintainer Johannes Sixt today. The 13-patch v8 series aims to fix cryptic failures in edge cases involving bare repos, missing worktrees, and .git directory launches. Today's discussion focused on several key patches:

- Patch 7/11 (centralized repository validation) is approved with minor polishing needed
- Patch 9/11 (Windows path handling) requires restructuring to properly handle path comparisons between Tcl and Git output
- Patch 10/11 (worktree discovery) needs changes to directory change behavior and error handling
- Patch 11/11 (repository picker subcommands) requires clarification on picker invocation timing

The maintainer's feedback continues the thread's pattern of careful attention to cross-platform correctness and maintainability. While the series still needs work, the technical direction appears sound with all major architectural questions resolved.

### Performance optimization for commit-reach.c ready

Kristofer Karlsson's performance optimization for `tips_reachable_from_bases()` in commit-reach.c has reached v3 with all review feedback addressed. The change replaces an O(C*T) linear scan with O(1) flag checks using the RESULT object flag, providing 4x-6x speedups for ref-filter operations like `for-each-ref --merged`. The v3 iteration adds comprehensive regression tests covering the duplicate-tips case that exposed a bug in the initial implementation, addressing Derrick Stolee's review request. With Jeff King's earlier concerns about the decoration API usage resolved by switching to object flags, this optimization now appears production-ready and likely to benefit operations in large repositories.

## In brief

**ALLOC_GROW macro refactoring** -- René Scharfe and Jeff King continue analyzing edge cases in the core memory macro, revealing dangerous behaviors with small integer types that could lead to buffer overruns or near-infinite allocations.

**"Today" means midnight in date parsing** -- A separate thread confirms successful integration of the "today as midnight" change with Tuomas Ahola's pending work on "noon"/"tea" time specifications, with a v4 series coming soon.

**UBSan CI failures fixed** -- Jeff King provided clean solutions for two edge cases found by EXPENSIVE tests: a memory leak in large patch handling and an integer overflow in UTF-8 verification of giant commit messages.

**Documentation typo fixes** -- Clinton Phillips submitted patches correcting duplicate words and misspellings in release notes and technical docs, while another patch fixes "accomodate" in a pack-objects.c comment.

**git stash show test coverage** -- Pushkar Singh added a test case verifying `git stash show --include-untracked` behavior, filling a gap in the existing test suite.

## On the radar

**Config syntax hint improvements** -- Harald Nordgren's v2 patch adding "did you mean" hints for common `git config` syntax errors appears merge-ready after addressing all review feedback, with no outstanding objections.