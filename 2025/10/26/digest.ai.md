# Git Mailing List Digest - 2025/10/26

**The day in brief.** October 26th saw moderate activity with 17 emails across 6 threads, featuring several technical discussions reaching resolution. The most notable developments include Jeff King's finalized fix for `**` pattern matching in `.gitignore` files, Lucas Seiki Oshiro's `git-repo-info` series nearing completion, and ongoing refinements to `git add -p`'s EOF handling. Benchmark data played a key role in validating optimization tradeoffs today.

## Notable threads

### `**` pattern matching fix finalized

Jeff King (Peff) completed his two-patch series fixing incorrect `**` pattern matching in `.gitignore` and `.gitattributes` files. The solution preserves Git's prefix-stripping optimization (which benchmarks show provides a 1.46x speedup for exact matches) while correctly handling edge case patterns like `foo**/bar`. The implementation now checks for complete matches first before adjusting pointers, then preserves one character of context during prefix stripping to maintain directory boundary awareness. Junio Hamano confirmed the approach after reviewing benchmark data showing the optimization's value outweighs any minor overhead from the fix.

### `git-repo-info` --all flag implementation

Lucas Seiki Oshiro's v3 series for the experimental `git-repo-info` command implements the `--all` flag interface as agreed in earlier discussion. The well-structured series first refactors field printing logic (receiving code quality suggestions from Eric Sunshine) before adding the new flag with thorough test coverage. The implementation prioritizes scripting safety by erroring out when `--all` is used with specific keys, addressing Junio's earlier concerns. With all major design questions resolved and only minor implementation feedback remaining, this feature appears ready for inclusion.

### `git add -p` EOF handling refined

The thread on `git add -p`'s exit behavior reached consensus that silent EOF exits (versus explicit 'q' command feedback) match Unix tool conventions and should remain as implemented. Junio Hamano incorporated René Scharfe's test improvement using `test_grep` instead of raw `grep` calls, verifying the interface exits immediately on EOF after showing the first file's diff. This concludes the refactoring work with no changes needed to the queued patches.

## In brief

**GPG test flakiness** -- Junio C Hamano proposes marking GPG2-requiring tests in t1016 as FLAKEY to work around intermittent CI failures with SHA-1 verification, while seeking a proper fix for the underlying issue.

**Wildmatch follow-up** -- Jeff King indicates he's posted a re-roll of a wildmatch bugfix for `**` pattern matching, with the patch previously marked for merging to 'next'.

## On the radar

**Test infrastructure gaps** -- The GPG test discussion revealed potential gaps in test coverage for multi-hunk/multi-file EOF cases in interactive commands, an area that may warrant future attention.