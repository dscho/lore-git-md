# Git Mailing List Digest — 2025/01/01

## The day in brief

A moderately busy New Year's Day in Git development saw 18 emails across 8 threads, with Jeff King (peff) driving much of the discussion around LeakSanitizer race conditions in threaded tests. The day's notable developments included a comprehensive solution to LSan false positives, a bugfix for object name resolution with curly braces, and routine release candidate validations for both mainline Git and Git for Windows.

## Notable threads

### LeakSanitizer race condition resolution

Jeff King dominated today's traffic with a six-patch series addressing persistent false positives in LSan (LeakSanitizer) output during threaded test execution. The solution represents a strategic shift from earlier attempts to prevent races through thread synchronization (now reverted) to a more robust approach of filtering known false positives in LSan's diagnostic logs. The series:

1. Fixes stress test output isolation (preventing concurrent runs from corrupting LSan logs)
2. Reverts an ineffective thread barrier in index-pack
3. Modifies LSan to report leaks without aborting
4. Simplifies the test infrastructure's leak counting
5. Tightens leak detection to focus on definitive markers
6. Adds specific filtering for the problematic `GetThreadStackTopAndBottom` race

The approach demonstrates Git's pragmatic handling of toolchain limitations while maintaining test reliability. Junio has not yet commented on the series, but the solution appears well-considered and minimally invasive.

### Object name resolution with curly braces

Pratyush Yadav contributed a bugfix addressing Git's mishandling of branch names containing unpaired curly braces (like `foo{bar`), a quirk dating back to 2006. The patch modifies `object-name.c` to only treat braces as special when they follow an `@` symbol, allowing proper resolution of paths like `foo{bar:README.md`. Junio engaged with follow-up questions about colon handling in commit searches, which the comprehensive test suite confirms remains unaffected. This is the kind of long-tail edge case that rarely surfaces but matters when it does.

### Race condition in object collision checking

Jeff King and Junio discussed a subtle race condition in Git's object collision detection during packfile operations, particularly when running concurrent `gc --prune=now`. The thread examined three race windows where temporary file disappearance could lead to incorrect success returns. Both agreed that explicit failure on tempfile disappearance provides better diagnostics than silent error masking, even if some races remain theoretically possible. The discussion reflects Git's careful balancing of correctness versus practicality in edge cases.

## In brief

Git for Windows 2.48.0-rc1 arrived with updated dependencies (cURL 8.11.1, MinTTY 3.7.7) and formalized Windows 7/8 installation blocking. Randall Becker confirmed successful NonStop platform builds for mainline Git's 2.48.0-rc1. Junio provided a crisp explanation of Git's branching model in response to a basic question about change visibility across branches. A test modernization patch replaced raw `test -f` checks with Git's helper functions in t7110-reset-merge.sh.

## On the radar

The thread synchronization utilities discussion from yesterday appears to be losing steam after Peff's closing remark questioning whether the series is needed at all. This may mark the end of that exploration unless new arguments emerge. The LSan series seems likely to progress given its comprehensive solution to a known CI pain point.