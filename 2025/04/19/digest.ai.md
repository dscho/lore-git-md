# Git Mailing List Digest — 2025/04/19

## The day in brief

A moderately active day with 6 emails across 4 threads, featuring Junio's weekly "What's cooking" report, continued debugging of wildcard pathspec tests, and a design discussion about `blame.ignoreRevsFile` behavior. The most notable development is Junio's pushback on a blame configuration patch, advocating for a more general solution to optional file handling.

## Notable threads

**Wildcard pathspec test debugging** -- Jayatheerth K's attempt to add a test case for `git commit`'s wildcard pathspec handling hit snags, with Lucas Seiki Oshiro identifying two implementation issues: an undefined `reset_git_repo` helper function and failing test comparisons. The test aims to verify behavior with special characters (*, ?, []) in filenames, building on earlier investigations into pathspec inconsistencies between commands. While the test isn't yet functional, its goal — validating whether wildcards in `git commit` properly match expected files — remains important for the ongoing pathspec consistency work.

**Blame ignoreRevsFile behavior debate** -- A patch proposing to make `blame.ignoreRevsFile` configuration tolerate missing files sparked a design discussion. The initial approach treated config-specified files more leniently than command-line ones, but Junio rejected this as too narrow a solution. He counter-proposed a unified `:(optional)` prefix syntax that could work across Git commands and configurations, arguing the problem deserves infrastructure-level treatment rather than command-specific exceptions. This thread may evolve into a broader discussion about optional file handling in Git's configuration system.

## In brief

**What's cooking report** -- Junio's weekly status update highlights recent changes including reftable backend improvements, Perl dependency reductions, and new machine-parsable output options. Several topics are queued for merging while others await further review.

**Performance test variables** -- Jeff King confirmed Johannes Schindelin's fix for `GIT_PERF_*` variable handling works but noted additional issues with default values in test scripts that will need future attention.