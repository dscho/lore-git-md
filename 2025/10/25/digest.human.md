# Git Mailing List Digest — 2025/10/25

**The day in brief.** A moderately active Saturday with 13 emails across 7 threads, featuring final approvals for two significant changes (atomic `git replay` updates and `git add -p` refactoring), ongoing documentation discussions about log formatting, and Junio's weekly "What's cooking" report. The most notable developments are the imminent merging of the atomic ref updates series and a clean simplification of interactive patch selection's exit handling.

## Notable threads

### Atomic ref updates in `git replay` approved

Junio Hamano has given final approval to the atomic reference updates series for `git replay`, marking the culmination of extensive review. The implementation now defaults to atomic operations with configurable output modes via `--ref-action` and `replay.refAction`. All major technical concerns — including atomicity verification, performance validation, and test coverage — have been resolved through multiple review rounds with Patrick Steinhardt, Phillip Wood, and others. The series awaits one final reroll to address minor style/typo nits before merging into 'next', representing the RFC-to-production transition for this feature.

### Geometric repack test race condition

Jeff King surfaced a test reliability issue in the geometric repacking series, where timestamp updates to the multi-pack-index during no-op repacks cause test failures. The discussion clarifies whether midx regeneration should be considered expected behavior in no-op cases, with proposed fixes ranging from making tests timestamp-insensitive to potentially fixing what might be a bug. This final polish issue in an otherwise approved series highlights the meticulous attention to test hygiene that characterizes Git's development process.

### `git log` format string documentation gaps

An ongoing discussion about `git log` format strings and notes display gained new clarity from René Scharfe and Junio Hamano. The thread revealed undocumented details about `--oneline`'s and confirmed limitations in notes formatting (no way to strip newlines from %N). Hamano noted the long-stalled effort to unify format languages between `git log` and `git for-each-ref`, while Scharfe suggested implementing named formats as actual format strings could improve documentation clarity. The exchange moved from problem identification toward potential solutions, with both immediate documentation improvements and deeper implementation changes now on the table.

### `git add -p` exit handling simplified

René Scharfe's two-patch series to clean up `git add -p`'s exit handling received quick approval from Junio. The changes separate 'q' (quit) command processing from 'd' (skip) commands and make EOF (Ctrl-D) trigger immediate exit. Junio expressed surprise these cleanups hadn't been noticed earlier in code he originally wrote, highlighting how even long-established interfaces benefit from ongoing refinement. The small but thoughtful changes improve internal consistency without affecting user-visible behavior, demonstrating Git's culture of incremental quality improvements.

## In brief

**Credential helper installation** — A build system patch adds standard `install` targets for libsecret and osxkeychain credential helpers in contrib/, following up on earlier Makefile cleanup work.

**What's cooking report** — Junio's weekly status update notes several bugfixes graduated to master, new topics in development (including diff dry-run cleanup and merge-base optimization), and significant ongoing work like Rust bindings generation and geometric repacking improvements.

**Interactive patch selection EOF handling** — A follow-up discussion considered whether EOF exits should echo 'q' for consistency, ultimately concluding silent exit matches Unix conventions better, especially in `interactive.singlekey` mode.