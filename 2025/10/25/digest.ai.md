# Git Mailing List Digest - 2025/10/25

**The day in brief.** A moderately active Saturday with 13 emails across 7 threads, featuring final approvals for two significant changes (atomic ref updates in `git replay` and `git add -p` exit handling), a detailed discussion about log formatting documentation, and Junio's weekly "What's cooking" report. The most notable developments are the imminent merging of the atomic ref updates series and the resolution of long-standing exit behavior quirks in interactive patch selection.

## Notable threads

### Atomic ref updates in `git replay` approved

Junio Hamano has given final approval to the atomic ref updates series for `git replay`, marking the culmination of extensive review. The implementation now defaults to atomic reference updates with configurable output modes via `--ref-action` and `replay.refAction`. All major technical concerns have been addressed, including atomicity verification, performance validation, and test coverage improvements. The series is in final polish stage, with Junio expecting one more small reroll to address minor nits before merging. This represents the transition from RFC to production-ready status for a feature that significantly improves the reliability of history rewriting operations.

### Geometric repack test race condition

Jeff King identified a test failure in the geometric repacking series related to multi-pack-index timestamp updates during no-op repacks. The test expects packfiles to remain unchanged but fails when midx regeneration updates timestamps. Jeff proposes two solutions: either make the test timestamp-insensitive (if midx regeneration is expected) or fix a potential bug (if no midx update should occur). This final polish issue in an otherwise approved series highlights the subtle edge cases that emerge when optimizing Git's storage layer.

### `git log` format string documentation

A detailed discussion continued about improving documentation for `git log` format strings and notes display. René Scharfe provided authoritative answers about format string equivalences, confirming that while `--oneline --notes` works as a practical solution, deeper limitations exist in notes formatting due to their free-form nature. Junio Hamano noted the long-discussed but inactive effort to unify format languages between `git log` and `git for-each-ref`. The thread moved from problem identification toward potential implementation improvements, with René suggesting that named formats could be reimplemented using the format string system they describe.

### `git add -p` exit handling refactor

René Scharfe's two-patch series to simplify exit handling in `git add -p` received quick approval from Junio Hamano. The changes separate 'q' (quit) command handling from 'd' (skip) commands and make EOF conditions (like Ctrl-D) trigger immediate exit. Junio expressed surprise these cleanups hadn't been identified earlier, noting he originally wrote this code before its C port. The small but logical improvements clarify internal behavior without affecting functionality, continuing Git's pattern of incremental code quality improvements.

## In brief

**Credential helper installation** -- A build system patch adds standard `install` targets to the libsecret and osxkeychain credential helper Makefiles, following the pattern used in other contrib/ directories.

**What's cooking** -- Junio's weekly status update highlights several bugfixes graduated to master, new topics in development (including diff dry-run cleanup and merge-base optimization), and significant ongoing work like Rust bindings generation and geometric repacking improvements.

## On the radar

**Geometric repack test fix** -- The race condition identified by Jeff King needs resolution to determine whether midx regeneration should be considered part of a "no-op" geometric repack.