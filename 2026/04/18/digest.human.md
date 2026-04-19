# Git Mailing List Digest - 2026/04/18 (Saturday)

**The day in brief.** A moderately active Saturday with 29 emails across 7 threads, featuring the completion of Lorenzo Pegorari's promisor repack series, continued discussion on test suite error handling, and a new revision walk option proposal. The most notable developments were the finalization of the promisor file preservation work and the resolution of an mksh shell behavior debate in the test suite.

## Notable threads

### Promisor file preservation during repacks finalized

Lorenzo Pegorari's GSoC project to preserve `.promisor` file content during repacks reached completion with the v6 series. The implementation now handles both regular and geometric repacks while maintaining debugging information in promisor files. Key improvements in this version include robust validation of reference names, proper handling of NULL returns from packfile parsing, and memory leak fixes. The series has comprehensive test coverage and has addressed all substantive feedback from reviewers including Tian Yuchen and Junio Hamano. With only minor documentation wording nits remaining, this work appears ready for integration.

### Test suite error handling debate continues

The discussion around enabling `set -e` error checking in Git's test suite saw significant progress as brian m. carlson confirmed an mksh shell bug in `t1410-reflog.sh`, resolving earlier disagreement with Jeff King (Peff) about whether the behavior was correct POSIX interpretation. Meanwhile, Jeff proposed extending the error-checking functionality via a `GIT_TEST_USE_SET_E` configuration option rather than limiting it to Bash 5+. Junio Hamano suggested moving to capability-based detection rather than version checks, though Peff noted the complexity of fully verifying shell behavior. The thread remains technically deep with multiple approaches being explored to make error detection both strict and portable.

### New revision walk option proposed

Mirko Faina proposed adding a `--reverse=before` option to revision walks, allowing reversal to happen before commit limiting rather than after. The patch would let commands like `git log --reverse --max-count=3` return the first 3 commits rather than the last 3. Tian Yuchen's review suggested more user-centric terminology and memory efficiency improvements, leading to productive discussion about implementation tradeoffs. Mirko acknowledged the feedback and plans to incorporate a sliding window approach in v2 to optimize memory usage during full history traversal.

## In brief

**gitk race condition fix** -- Johannes Sixt responded to Michael Warmuth-Uhl's patch addressing a "can't read 'treediffs'" error, proposing an alternative queue-clearing approach that currently fails on recursive execution but may offer better performance than the original waiting solution.

**Partial clone optimizations** -- Elijah Newren's series optimizing `git cherry` and `git grep` for partial clones is now ready for merging after removing an extraneous design document and fixing minor documentation syntax.

**Test suite robustness fix** -- SZEDER Gábor corrected tilde expansion behavior in `t6112-rev-list-filters-objects.sh` by quoting grep patterns, preventing potential shell expansion issues and Bash segfaults in versions 3.2-5.0.

## On the radar

**`the_repository` removal effort** -- Elijah Newren's reviews of Patrick Steinhardt's `setup.c` refactoring identified remaining global state that should be addressed to fully support multiple worktrees, suggesting this architectural work isn't quite complete despite the series being ready for merging.