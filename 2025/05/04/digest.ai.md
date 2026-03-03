# Git Mailing List Digest — 2025/05/04

## The day in brief

A moderately active Sunday with 12 emails across 5 threads, featuring a mix of bug reports, documentation improvements, and platform compatibility discussions. The standout items include a proposal to deprecate Windows support for `git svn`, a thoughtful challenge to a merge visualization proposal, and a series improving `git send-email`'s FQDN validation and OAuth documentation.

## Notable threads

**Windows `git svn` maintenance concerns** — Johannes Schindelin responds to Arnoud Berendsen's report of Windows-specific `git svn` failures by acknowledging the feature's precarious maintenance state in Git for Windows. The discussion reveals no current developers are investing time in Windows-specific fixes, with Schindelin suggesting WSL as a workaround and linking to an existing GitHub issue about potentially dropping Windows support. This continues an ongoing conversation about the feature's sustainability on Windows, where Perl-based components have proven particularly problematic.

**Merge visualization design challenge** — Johannes Sixt raises a fundamental design question in response to Britton Kerin's proposal for 3-pane merge visualization. Sixt points out that while 3-pane views work well for 2-parent merges (base/ours/theirs), they don't generalize to higher-order merges with multiple parents. The thoughtful critique moves the discussion forward by identifying a key design consideration that would need addressing in any implementation, while acknowledging the potential usefulness for common cases.

**`git send-email` improvements** — Aditya Garg submits a three-patch series combining technical fixes and documentation updates. The first patch tightens FQDN validation in `git-send-email.perl` to comply with RFC1035 standards, while the remaining patches significantly expand OAuth configuration documentation for modern email providers. The series receives minor formatting feedback but appears technically sound, with the documentation updates particularly valuable for users navigating modern email requirements. Kristoffer Haugsbakk suggests a commit message formatting improvement, which Garg agrees to incorporate in a potential v2.

## In brief

**Unnecessary diff warnings in `git show -s`** — Junio Hamano reports that `git show -s` unnecessarily checks rename detection limits even when no diff output is generated. Jeff King responds with a previously shelved patch that would skip diff computation entirely in this case.

**macOS signal handling compatibility** — Torsten Bögershausen reports a compilation failure on older macOS systems related to `NOT_CONSTANT` and `sigfillset()`, proposing several potential solutions including a possible revert of the problematic commit.