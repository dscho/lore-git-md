# Git Mailing List Digest — 2025/04/19

**The day in brief.** A moderately active Saturday with 6 emails across 4 threads, featuring Junio's weekly "What's cooking" report, a proposed fix for `blame.ignoreRevsFile` behavior that sparked design discussion, and continued debugging of wildcard pathspec test cases. The most consequential development is Junio's pushback on the blame ignore-file patch, suggesting a more general solution is needed.

## Notable threads

### Junio's weekly development update

Junio Hamano's "What's cooking" report outlines recent progress across Git's development branches. Several cleanups and test improvements have graduated to `master`, including removal of unused variables in `environment.h` and Perl dependency reductions. The reftable backend saw significant work with recursive merge strategy remnants being removed. New topics under consideration include documentation for disabling hooks via `core.hooksPath` and optimizations for pack-objects to avoid cruft packs in MIDX. Ongoing work includes meson build system improvements, continued Perl dependency reduction, and object-file subsystem cleanups. The report serves as both a progress snapshot and call for review on pending changes.

### Blame ignore-file behavior debate

A patch proposed making `blame.ignoreRevsFile` more lenient when files specified in config don't exist — currently Git exits in this case, which prevents setting global defaults like GitHub's `.git-blame-ignore-revs`. The initial solution treated config-specified files differently from command-line ones. Junio rejected this approach as too blame-specific, proposing instead a unified `:(optional)` prefix syntax that could work across all Git commands. This positions the issue as requiring infrastructure-level changes rather than a command-specific fix, likely delaying resolution while a more general solution is designed.

## In brief

**Wildcard pathspec test debugging** -- Jayatheerth K's test case for `git commit` wildcard handling is failing comparison checks, with Lucas Seiki Oshiro identifying an undefined `reset_git_repo` function and suggesting verbose test output for debugging. This continues investigation into pathspec inconsistencies.

**Performance test variables** -- Jeff King confirmed Johannes Schindelin's fix for `GIT_PERF_*` variable handling works but noted remaining issues with `GIT_PERF_REPEAT_COUNT` defaults, part of broader test environment variable challenges.