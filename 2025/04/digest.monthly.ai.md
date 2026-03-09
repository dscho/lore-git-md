# Git Mailing List Digest — 2025 April

**The month in brief.** April 2025 saw significant architectural progress across Git's codebase with 1,326 emails across 410 threads. The month was marked by several major milestones: the completion of Elijah Newren's multi-year effort to remove merge-recursive in favor of merge-ort, Patrick Steinhardt's extensive reftable API overhaul, and the finalization of Perl test dependency removal. Other notable developments included Windows/ARM64 support landing, object storage subsystem reorganization, and ongoing debates about Change-ID standardization. The month also saw steady progress on build system modernization, test infrastructure improvements, and documentation updates.

## Key developments

**Merge-recursive removal completed**  
Elijah Newren's multi-year effort to replace Git's legacy merge-recursive implementation with merge-ort reached its conclusion this month. The final series systematically converted remaining callers (checkout, merge, sequencer) to use merge-ort, added final feature parity (diff algorithm support), then removed merge-recursive.[ch] and associated test infrastructure - deleting over 5,000 lines of code. Junio Hamano suggested reframing the removal documentation to emphasize bug elimination rather than using the original "debug" terminology. This architectural simplification represents a major milestone in Git's merge machinery evolution, with merge-ort having been the default strategy since Git 2.33.0.

**Reftable API overhaul**  
Patrick Steinhardt's extensive work on the reftable subsystem standardized its public API and improved external usability. The changes included renaming core structures (`reftable_reader` → `reftable_table`), exposing block-level iteration for verification, and carefully separating public/private interfaces. This enables future consistency checking in Git's reftable backend while making the library more suitable for projects like libgit2. The series showed meticulous attention to API boundaries, with comprehensive test coverage accompanying each change. Review feedback from Justin Tobler and Karthik Nayak focused on minor naming and interface refinements before final approval.

**Object storage subsystem reorganized**  
Patrick Steinhardt's significant refactoring of Git's object storage code split object-file.c into logical components (object-store.c, read-cache.c, etc.) while eliminating global state. The changes touched 124 files with 2,296 insertions and 2,224 deletions, establishing clean boundaries needed for future pluggable storage backends. The remaining object-file.c now focuses specifically on loose object handling. This work is part of the broader `the_repository` removal effort and received positive feedback from Junio Hamano, who approved the current approach as safe for merging despite initial questions about virtual objects storage location.

**Windows/ARM64 support lands**  
Dennis Ameling and Johannes Schindelin's comprehensive series enabled Windows/ARM64 support, addressing compiler compatibility, memory allocator selection (with discussion about potentially upstreaming mimalloc), MSVC requirements, and stack overflow prevention specific to ARM64 builds. The cleanly structured changes received quick approval from Junio Hamano with only minor style adjustments needed. This work positions Git for the expanding ARM64 ecosystem while maintaining compatibility with existing Windows deployments.

**Build system standardization**  
Multiple build system improvements reached completion this month. Ramsay Jones aligned Makefile and Meson builds with Cygwin-specific fixes, while Karthik Nayak implemented header checking in Meson builds. Patrick Steinhardt wired up Git's performance tests to the Meson build system, marking steady progress in Git's modernization efforts. These changes represent important milestones in Git's multi-year Meson adoption, bringing static analysis features to parity with the traditional build system.

## In brief

**Perl test dependency removal** -- Patrick Steinhardt's series eliminated Perl as a mandatory dependency for Git's test suite, allowing 97% of tests (30,342 of 31,358) to run without Perl and achieving a 10MB size reduction for Git for Windows.

**UTF-8 handling in ref operations** -- A critical bug addressed crashes when processing repositories containing Unicode refnames by fixing the packed-ref iterator's comparison logic to properly handle UTF-8 characters.

**Change-ID standardization debate** -- Martin von Zweigbergk's cross-tool collaboration generated extensive discussion about implementation semantics, revealing divides between Git's content-addressable design and practical metadata tracking needs.

**Batched reference updates** -- Karthik Nayak's series added `--batch-updates` flag for `git update-ref` with partial failure support across files, packed, and reftable backends, providing groundwork for robust reference operations.

**Integer parsing safety** -- Patrick Steinhardt hardened integer handling in Git's parse-options infrastructure with comprehensive error handling for overflow/underflow cases and build-time validation.

**MIDX/cruft pack optimization** -- Taylor Blau's series introduced configurable control over whether cruft packs must be included in the MIDX via `repack.midxMustContainCruft`.

**Email infrastructure improvements** -- Aditya Garg and Julian Swagemakers added OAuth2 support to `git send-email` using Git's credential helper system and finalized Outlook Message-ID handling.

**Documentation modernization** -- Jean-Noël Avila converted `git-reset`, `git-rm`, and `git-mv` man pages to AsciiDoc with special character handling fixes as part of the ongoing synopsis-style conversion.

**Bundle creation optimization** -- Karthik Nayak's O(N^2) to O(1) optimization for bundle ref deduplication showed 6x speedups in 100k-ref repositories.

**Stash behavior clarification** -- Discussion revealed that files popped from stash are permanently lost when aborting a failed cherry-pick, potentially prompting documentation improvements.

**Perforce test modernization** -- Tests were updated to use `git show-ref --verify` instead of fragile `grep` patterns for robust tag verification.

**Blame ignore-file behavior** -- Proposed fixes sparked design discussion about a general `:(optional)` path prefix solution rather than command-specific changes.

**MyFirstContribution updates** -- The tutorial was modernized to demonstrate current practices like `repo_config()` usage and proper repository parameter handling.

## Looking ahead

Several topics are poised for continued development in May:

- The Change-ID discussion may see renewed attention as participants weigh different approaches to tracking commit evolution.
- Rustification efforts could resurface as platform support questions remain unresolved.
- Windows-specific issues including wildcard handling and asyncio problems need further investigation.
- Documentation improvements around stash behavior during operation aborts may emerge from recent discussions.
- The path-based delta compression series remains in "What's Cooking" awaiting deeper maintainer review.