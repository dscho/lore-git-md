# Git Mailing List Digest — 2025/02/22

## The day in brief

A moderately active day with 10 emails across 8 threads, featuring ongoing technical discussions about documentation structure, shell function recognition, and performance optimizations. The most notable developments include a maintainer-approved fix for `git check-mailmap`'s NULL pointer handling and a new report about `git gc --auto` failing to clean large reflogs.

## Notable threads

**Submodule merge documentation debate reaches compromise**  
The discussion about documenting submodule merge behavior sees progress as Junio Hamano proposes a middle path between full duplication and complex AsciiDoc attributes. Following Lucas Seiki Oshiro's argument that the current approach has remained stable for three years, Junio suggests keeping the full warning text in the 'ort' section while adding a brief cross-reference in the 'recursive' section. This solution addresses the transitional nature of the recursive-to-ort migration while avoiding maintenance pitfalls. The thread demonstrates Git's careful approach to documentation structure, balancing clarity against future maintainability.

**Shell function recognition improvements face regex challenges**  
Johannes Sixt provides detailed technical feedback on Moumita's v2 patch for Bash function pattern matching in `userdiff.c`. The review identifies regex syntax issues causing test failures in `t4018-diff-funcname.sh` and clarifies that certain pattern requirements stem from the userdiff driver framework's constraints rather than being intentional design choices. Johannes suggests simplifying the approach by capturing entire function headers and provides specific recommendations for handling variable expansion patterns. The thread continues Git's tradition of thorough pattern-matching reviews, with the immediate blocking issue being the test suite regression.

**`git check-mailmap` NULL pointer fix approved**  
Junio Hamano reviews and approves the second iteration of a bugfix addressing a segfault when processing email-only inputs in `git check-mailmap`. The minimal solution changes a NULL name pointer to an empty string in `builtin/check-mailmap.c` while preserving existing mailmap resolution behavior. A new test case verifies email-only input works with mailmap address mappings. This straightforward fix, likely to be queued soon, demonstrates Git's preference for targeted solutions over broader interface changes when addressing specific bugs.

## In brief

Jeff King spots a type mismatch in Patrick Steinhardt's `the_repository` removal series, noting `rerere_last_used_at()` should use `timestamp_t` rather than `int` for timestamp storage. The issue doesn't affect the core refactoring but would cause type safety problems.

Junio Hamano and Christian Couder continue discussing `git bisect` optimization thresholds, clarifying that the 10,000-commit cutoff preserves correctness by ensuring nr/1024 doesn't truncate to zero during final bisection stages.

Audrey Dutcher receives guidance on documenting OpenBSD build issues with clang 19's stricter warnings, particularly around properly capturing the temporal aspect of compiler behavior changes in commit messages.

## On the radar

John Giorshev reports that `--filter=blob:none` silently falls back to full downloads with servers lacking filtering support, proposing to make this a hard error. The change would break backward compatibility, requiring discussion about opt-in approaches.

Markus Gerstel discovers `git gc --auto` failed to clean a 180MB reflog over six years in a standard-configured repository, suggesting a long-standing edge case in the gc heuristics that may need investigation.