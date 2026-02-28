# Git Mailing List Digest — 2025/01/01

## The day in brief

A moderately busy New Year's Day with 18 emails across 8 threads, featuring ongoing technical work on thread synchronization, leak detection, and object handling. The most notable developments include Jeff King's comprehensive solution for LSan race conditions in threaded tests and a fix for object name resolution with curly braces in branch names. Windows users saw the release of Git for Windows 2.48.0-rc1 with ARM64 support.

## Notable threads

### LSan race condition solutions

Jeff King (peff) dominated today's traffic with a six-patch series addressing LeakSanitizer (LSan) race conditions in threaded test execution. The series presents a novel approach to handling false positive leak reports by filtering LSan output rather than attempting to fix the underlying race in the sanitizer's stack unwinding code. Key changes include:

- Isolating LSan outputs for stress tests (patch 1)
- Reverting an ineffective thread synchronization attempt (patch 2)
- Transitioning to log-based leak detection (patches 3-5)
- Adding specific filtering for known false positives (patch 6)

The solution demonstrates Git's pragmatic approach to toolchain limitations while maintaining test reliability. The series appears ready for merging after thorough discussion of the tradeoffs between diagnostic quality and implementation complexity.

### Object name resolution with curly braces

A bugfix from Eric Sunshine addresses a long-standing issue (dating to 2006) where Git would fail to resolve object names containing unpaired curly braces in branch names. The patch modifies parsing logic in `object-name.c` to only treat curly braces as special when they follow an '@' character, allowing commands like `git cat-file -p foo{bar:README.md` to work as expected. 

Junio Hamano verified the fix maintains backward compatibility with existing `@{...}` syntax and colon handling in commit searches. The patch includes comprehensive tests covering both the new behavior and edge cases, demonstrating careful attention to backward compatibility.

### Thread synchronization utilities

An ongoing discussion about thread synchronization utilities continued today, focusing on patch 3/5 in a series introducing optional barrier types for LSan builds. The debate centers on platform/build matrix complexity, error handling philosophy (silent failure vs explicit checks), and whether the approach is even needed. Jeff King hinted the series might be avoidable entirely, suggesting the discussion may pivot toward alternative solutions.

## In brief

Git for Windows 2.48.0-rc1 was released with updated dependencies (cURL, MinTTY, MSYS2 runtime) and formalized Windows 7/8 installation blocking. The release notably includes ARM64 builds for the first time. Junio confirmed successful builds on NonStop for the upcoming v2.48.0-rc1. A test modernization patch replaced direct shell file checks with Git's test helpers in t7110-reset-merge.sh for better debuggability. Junio provided a concise explanation of Git's branching model in response to a fundamental question about branch visibility.

## On the radar

The thread synchronization utilities discussion appears to be at a crossroads, with Jeff King suggesting the barrier approach might not be the right solution after all. Watch for either a pivot in that series or its potential withdrawal in favor of alternative approaches to handling thread races in LSan builds.