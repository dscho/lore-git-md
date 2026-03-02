# Git Mailing List Digest — 2025/04/12

## The day in brief

A moderately active day with 29 emails across 9 threads, featuring continued refinement of documentation standards, resolution of a pathspec matching bug, and ongoing discussions about patch series tracking. The documentation modernization effort reaches a milestone with the completion of man page conversions for `git-reset`, `git-rm`, and `git-mv`, while a subtle but impactful wildcard matching issue in `git add` receives both a fix and new test coverage.

## Notable threads

### Documentation standardization reaches milestone

Jean-Noël Avila's 7-patch series converting `git-reset`, `git-rm`, and `git-mv` man pages to AsciiDoc format reached version 2, addressing all feedback from the initial review except for one remaining TROFF rendering quirk. The series standardizes synopsis formatting, special character handling, and triple-dot notation across these commands while updating both documentation and built-in help text. A final patch adds explicit character markup rules to CodingGuidelines, completing the standardization effort. The changes demonstrate meticulous attention to documentation presentation details, even for edge cases in the publishing pipeline.

### Pathspec matching bug resolved

After thorough analysis by multiple contributors, Jeff King proposed a solution to a long-standing wildcard matching issue in `git add`. The bug caused pathspecs like `f*` to incorrectly prioritize literal filenames matching the pattern (e.g., a file named `f*`) over wildcard expansion. The fix modifies `do_match_pathspec()` in dir.c to properly handle these edge cases while maintaining backward compatibility. Jayatheerth K followed up with new test coverage in t3706 that verifies the corrected behavior for both escaped and unescaped patterns. The thread progressed from initial report through technical analysis to a complete solution in under 24 hours.

### Alternative proposal for patch series tracking

Theodore Ts'o proposed a "Patch Set ID" mechanism as an alternative to Change-IDs for tracking patch series evolution. The design introduces a `--series` flag for `git commit` that establishes explicit parent-child relationships between commits, with the ID persisting through rebases and email operations. This approach differs from Change-IDs by making series membership opt-in rather than automatic, potentially addressing some of the reachability concerns raised in earlier discussions. The proposal would require changes to several core commands but offers a more Git-native solution for tracking patch evolution.

## In brief

Philippe Blain's performance test fixes expanded to three patches, with the new addition correcting documentation in p7821-grep-engines-fixed.sh while the scalar test fix from v1 remains ready for merging. Junio Hamano weighed in on the Change-ID discussion, suggesting technical approaches to mitigate commit rewriting concerns. A Perforce integration test improvement finalized after six iterations, replacing fragile grep checks with direct `git show-ref --verify` calls. Jeff King provided architectural feedback on Patrick Steinhardt's object-file reorganization, questioning whether certain functions conceptually belong in the index subsystem. Lucas Seiki Oshiro clarified that `git stash -k`'s behavior of preserving staged changes is working as documented, though potentially surprising to some users.

## On the radar

The documentation standardization effort now turns to resolving the remaining TROFF rendering quirk with special characters in bold spans, which may require deeper toolchain investigation. Patrick Steinhardt's object-file reorganization series awaits potential adjustments based on Jeff King's feedback about subsystem boundaries. The patch series tracking discussion continues to evolve with Theodore Ts'o's new proposal, which may influence future workflow tooling.