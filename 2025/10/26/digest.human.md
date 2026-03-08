# Git Mailing List Digest - 2025/10/26

**The day in brief.** A moderately active Sunday with 17 emails across 6 threads, featuring progress on several technical fronts. Key developments include the resolution of a long-standing pattern matching bug, finalization of the `git-repo-info` interface, and continued refinement of `git add -p` behavior. Performance benchmarks helped validate optimization tradeoffs in one thread, and test infrastructure issues surfaced in another.

## Notable threads

### `git-repo-info` interface finalized

Lucas Seiki Oshiro's v3 series for the experimental `git-repo-info` command implements the `--all` flag as agreed in earlier discussion, with mentorship from Karthik Nayak and Patrick Steinhardt. The two-patch series first refactored field printing logic into a dedicated function, then added the new flag with strict error handling when combined with specific keys - addressing Junio's scripting safety concerns. Eric Sunshine provided focused review on implementation details, suggesting API improvements to make the printing function more self-contained. The thread shows a feature progressing from design discussion to polished implementation, now likely ready for inclusion.

### Pattern matching optimization validated

Jeff King's benchmark-driven analysis resolved the debate around Git's pattern matching optimization in `dir.c`, showing the prefix-stripping approach provides significant speedups (1.46x for exact matches) while his "minusone" solution correctly handles `**` edge cases. The final two-patch series reorganizes the code for clarity and implements the one-character context preservation, with Junio confirming the approach after seeing performance data. This demonstrates Git's careful balance between optimization and correctness, with the thread progressing from bug report to benchmark-backed solution.

### `git add -p` exit behavior clarified

The refactoring of `git add -p`'s exit handling reached completion with consensus that silent EOF exits match Unix conventions, despite differing from explicit 'q' command behavior. Junio incorporated René Scharfe's improved test case using `test_grep` instead of raw `grep`, finalizing the testing strategy for this edge case. The exchange documented the intentional design choice while highlighting potential gaps in multi-stage EOF test coverage, though no code changes were needed to the already-queued patches.

## In brief

**Test infrastructure GPG issues** -- Junio proposes marking flaky GPG-related tests in t1016 as FLAKEY while investigations continue into SHA-1 verification inconsistencies between CI and local environments.

**Wildmatch double-asterisk fix** -- Jeff King indicates a re-roll is posted for a wildmatch subsystem fix addressing `**` pattern matching edge cases, with the patch previously marked for merging to 'next'.

## On the radar

**GPG test flakiness** -- The intermittent CI failures in SHA-1 verification tests remain unresolved, with root cause analysis still needed beyond the temporary FLAKEY marking.