# Git Mailing List Digest - 2026/05/06 (Wednesday)

**The day in brief.** A moderately busy Wednesday with 32 emails across 15 threads, featuring significant progress on several fronts. Key developments include the finalization of the `--max-count-oldest` feature for revision walks, a comprehensive RFC series adding merge support to `git history`, and resolution of long-standing git-gui startup issues. The day also saw new bug reports and documentation improvements.

## Notable threads

### `--max-count-oldest` feature finalized

The `--max-count-oldest` feature, which shows the oldest N commits in a revision walk, has reached its final form after seven iterations. Johannes Sixt's review prompted Mirko Faina to make several final improvements: clearer man page wording contrasting the new option with `--max-count`, standardized error message formatting, and moving the counting logic into `get_revision_internal()` to properly handle `--boundary` and other options. The feature maintains its optimized O(K) space and O(N) time characteristics while now fitting cleanly into the revision walker's architecture. With all substantive feedback addressed, this appears ready for final submission and merging.

### git-gui startup sequence overhaul

Shroom Moo's v6 patch series fixing git-gui's handling of bare repositories and missing worktrees has reached consensus after extensive discussion. The changes restructure the startup sequence to properly detect repository state before Tcl initialization, disable visualization features when no worktree is available, and handle environment variables more robustly. Johannes Sixt and Mark Levedahl's detailed reviews ensured the solution maintains backward compatibility while fixing the 2019 regression in `--show-toplevel` behavior. The series now appears ready for merging, resolving a long-standing pain point for users working with bare repositories or complex worktree setups.

### RFC: Merge support for `git history`

Johannes Schindelin proposed an RFC series adding merge commit support to `git history`, the new history-rewriting command introduced in Git 2.54. The five-patch series implements replay of two-parent merges using a three-way merge composition approach (R=original parents' auto-merge, O=original merge, N=rewritten parents' auto-merge). The implementation includes thorough test coverage via a new `test-tool historian` helper that creates complex merge scenarios declaratively. While octopus merges and merge reverts remain out of scope, this RFC represents a significant step forward for the command's usability in merge-heavy workflows.

## In brief

**Refs backend consolidation** -- Toon Claes confirmed approval of Karthik Nayak's v4 series standardizing object validation and peeling logic across reference backends, leaving only minor stylistic refinements before merging.

**Windows large objects CI fix** -- Johannes Schindelin noted Patrick Steinhardt's diagnosis of a CI build failure caused by glibc's C11 `_Generic` usage conflicting with clang's C99 checking on Ubuntu 26.04 runners.

**`fetch --deepen` regression fix** -- Samo Pogačnik submitted the final patch fixing a regression where `--deepen` could incorrectly convert full clones to shallow ones, with test coverage confirming the fix.

**Terminal output edge cases** -- Mikael Magnusson and René Scharfe continued discussing edge cases in ANSI clear sequence handling, revealing tension between progress cleanup and attribute management needs.

**`git maintenance` remote failures** -- Phillip Wood identified the root cause of intra-repository prefetch failures in multi-remote setups, suggesting `skipFetchAll` as a temporary workaround.

**Documentation typos** -- Andrew Kreimer submitted a mechanical patch fixing spelling errors across multiple files using the `codespell` tool.

**`reflog expire` output fix** -- Sven Weiland reported a minor inconsistency in `--dry-run` output wording that doesn't match Git's standard "would" prefix convention.

**`add -p` word-diff view** -- A new patch introduced a 'w' command to show hunks in word-diff format, particularly useful for reviewing changes in structured text files.

## On the radar

**RHEL 6 build failures** -- The discussion continues about OpenSSL conflicts on this unsupported platform, with D. Ben Knoble suggesting the reporter use `git bisect` to identify the problematic change themselves.

**`git diff --word-diff` whitespace bug** -- Vincent Lefevre reported incorrect highlighting of whitespace-only changes, showing entire lines marked as changed rather than just the whitespace differences.