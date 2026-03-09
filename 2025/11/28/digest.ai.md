# Git Mailing List Digest — 2025/11/28

**The day in brief.** A moderately busy day with 28 emails across 11 threads, featuring significant progress on the `git replay --revert` feature, multiple bugfixes for `git last-modified`, and ongoing documentation improvements. The most notable developments include architectural discussions about revision range parsing and resolution of cross-platform issues in recent optimizations.

## Notable threads

### `git replay --revert` revision parsing refined

The discussion around `git replay --revert` reached new technical depth as maintainers worked through edge cases in revision range handling. Junio Hamano and Elijah Newren debated how to properly parse complex revision specifications like `topic~5 topic~3..topic` when supporting disconnected commits. The conversation revealed that the current implementation processes all ancestors of listed commits rather than just the commits themselves, contrary to initial assumptions. 

Key takeaways include:
- Negative ref handling (`^topic~7`) presents parsing challenges that may require a new revision walking flag
- Ancestry-path queries naturally align with connected commit workflows
- The team is balancing flexibility against implementation complexity while maintaining consistency with other Git commands

The thread shows the careful consideration being given to both backward compatibility and future extensibility in Git's revision parsing architecture.

### Cross-platform fixes for `git last-modified`

Two separate bugfix threads addressed issues in the recently merged `git last-modified` optimization series. Toon Claes confirmed the root cause of test failures on s390x and NonStop systems was a broader algorithmic problem in merge commit handling, not just an endianness issue. A second fix corrected incorrect memory initialization in the command's bitmap handling, with Jeff King suggesting potential architectural improvements to prevent similar bugs.

The fixes maintain the optimization's 2.5x-111.7x performance gains while correcting:
- Merge commit parent ordering in the generation-sorted priority queue
- Bitmap clearing operations that were improperly sized
- Cross-platform test failures in t8020-last-modified.sh

### `git-pull` documentation typo fixed

A straightforward but instructive documentation thread saw Stefan Rieche correct a command syntax typo in git-pull.adoc, changing "git --rebase abort" to "git rebase --abort". The discussion evolved into a lesson on Git project conventions as Kristoffer Haugsbakk and Junio Hamano provided guidance on commit message formatting and author identity consistency. The patch went through two iterations to align with project norms before receiving maintainer approval.

## In brief

**`git replay` empty commit handling** -- Phillip Wood's patch to make `git replay` drop commits that become empty during replay received positive feedback from both Junio Hamano and Elijah Newren, who agreed it matches `rebase`/`cherry-pick` behavior.

**German translation correction** -- Sören Krecker confirmed "Revision" is the proper German translation for generic revision errors in `git rev-parse --verify`, addressing brian m. carlson's earlier report about incorrect use of "Commit".

**Zsh test compatibility** -- Two minimal patches from Anders Kaseorg fixed test suite issues when run under zsh's POSIX-compatible mode, addressing a hanging SOCKS server test and incorrect string comparison in reftable tests.

**Reference backend URI proposal** -- A new environment variable `GIT_REF_URI` was proposed to specify reference backend type and location via URI format (e.g., `files:/path/to/refs`), continuing efforts to make ref storage more configurable.

**Help message consistency** -- Kristoffer Haugsbakk updated branch-related error messages to use `git help` instead of `man` for manual page references, improving platform compatibility and UI consistency.

## On the radar

**`git fetch` SIGSEGV report** -- Justin Su reported a segmentation fault during submodule operations on ARM64, though initial analysis by brian m. carlson suggests repository corruption may be a factor. The thread awaits more detailed diagnostics.