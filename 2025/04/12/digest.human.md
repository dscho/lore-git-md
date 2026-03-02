# Git Mailing List Digest — 2025/04/12

**The day in brief.** A moderately active Saturday with 29 emails across 9 threads, featuring ongoing documentation modernization, a pathspec matching bugfix, and continued discussion about patch series tracking. The most notable developments include a finalized test improvement for Perforce integration and a v2 documentation series addressing special character handling.

## Notable threads

### Documentation modernization reaches v2

Jean-Noël Avila's documentation standardization effort continues with a 7-patch v2 series converting `git-reset`, `git-rm`, and `git-mv` man pages to AsciiDoc format. The series builds on prior work standardizing command documentation, with particular attention to special character handling in synopses. Key improvements include:

- Fixed synopsis analysis logic for backslashes and asterisks
- Proper formatting of triple-dot notation in `git-mv`
- Added markup guidance for special characters in CodingGuidelines
- Updates to both documentation and built-in help text

The series resolves all previously identified issues except for a remaining TROFF rendering quirk with special characters in bold spans, which requires deeper toolchain investigation. The changes demonstrate meticulous attention to both content and presentation details.

### Pathspec matching bug identified and fixed

A multi-email technical discussion culminated in a fix for `git add`'s wildcard pathspec handling when literal filenames match the pattern (e.g., a file named 'f*'). Jeff King (peff) identified the root cause in `do_match_pathspec()`'s interaction with the `nowildcard_len` optimization. The fix ensures wildcards expand consistently while maintaining backward compatibility - literal matches can still be targeted by escaping wildcards. Jayatheerth K followed up with test cases validating the behavior for both escaped (`f\*\*`) and unescaped (`f*`) patterns.

### Perforce test improvements finalized

After six iterations of review, a test improvement for Perforce integration tests has reached its final form. The patch replaces fragile `grep`-based tag verification in `t/t9811-git-p4-label-import.sh` with direct ref checks using `git show-ref --verify`. The changes, which received thorough review from Junio Hamano and Eric Sunshine among others, now include:

- Complete removal of `git tag | grep` patterns
- Added negative test case checking for absence of TAG_F1_ONLY
- Fixed original issue with lost pipe exit codes
- Removal of legacy `git tag` commands

This exemplifies Git's rigorous test improvement process, where even verification methods undergo multiple review rounds to ensure robustness.

## In brief

Philippe Blain's performance test fixes expanded to three patches, adding documentation corrections to p7821-grep-engines-fixed.sh after addressing test prerequisite handling and scalar clone behavior in detached HEAD state. The scalar test fix has already received positive review as straightforward and pragmatic.

Junio Hamano weighed in on the Change-ID discussion, suggesting technical approaches to mitigate commit rewriting concerns by making predecessor links non-participatory in reachability or using Git's `--filter` capability. Theodore Ts'o later proposed an alternative "Patch Set ID" mechanism with explicit `--series` flag for commit grouping.

A `git stash -k` behavior clarification confirmed that preserving staged changes while stashing only unstaged changes is working as documented, with workarounds provided using `git cherry-pick -m 2` to access working tree changes independently.

Patrick Steinhardt's object-file reorganization drew feedback from Jeff King questioning whether certain index-related functions conceptually belong in read-cache.[ch], highlighting subtle subsystem boundary questions in the refactoring effort.

## On the radar

The documentation toolchain's remaining TROFF rendering quirk with special characters in bold spans may warrant follow-up investigation, though it doesn't block the current series. The pathspec matching fix appears ready pending final test case review, while the patch series tracking discussion continues to explore alternative implementations.