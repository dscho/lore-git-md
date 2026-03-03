# Git Mailing List Digest — 2025/05/04

## The day in brief

A moderately active Sunday with 12 emails across 5 threads, featuring a mix of bug reports, documentation improvements, and platform compatibility discussions. The most notable developments include a proposal to deprecate Windows support for `git svn`, a thoughtful critique of merge visualization proposals, and a series improving `git send-email`'s FQDN validation and OAuth documentation.

## Notable threads

**Windows `git svn` maintenance concerns** — Johannes Schindelin responds to Arnoud Berendsen's report of Windows-specific `git svn` failures by acknowledging the feature's precarious maintenance state in Git for Windows. The discussion reveals no current developers willing to invest in fixes, with Schindelin suggesting WSL with Linux Git as a workaround and linking to an existing GitHub issue about potentially dropping Windows support. This continues an ongoing conversation about the feature's reliability on Windows, where path stripping errors and Perl warnings in Git.pm have become persistent issues.

**Merge visualization design challenge** — Johannes Sixt raises a fundamental design question about Britton Kerin's proposed 3-pane merge visualization, pointing out that while the approach works well for 2-parent merges (base/ours/theirs), it doesn't generalize to higher-order merges with multiple parents. The thoughtful critique moves the discussion forward by identifying a key limitation that any implementation would need to address, while acknowledging the potential usefulness for common cases. This exchange highlights the careful consideration given to UI design decisions in Git's tooling.

**`git show -s` optimization opportunity** — Junio Hamano and Jeff King discuss unnecessary diff computation in `git show -s`, where Git still checks rename detection limits despite explicitly suppressing diff output. King surfaces an old patch that would skip diff computation entirely when both the output format is DIFF_FORMAT_NO_OUTPUT and always_show_header is set. The small but focused change (4 lines in log-tree.c) demonstrates Git's attention to avoiding wasteful computation, even in edge cases.

## In brief

**`git send-email` improvements** — Aditya Garg sends a 3-patch series combining technical and documentation updates. The first patch tightens FQDN validation in `git-send-email.perl` to comply with RFC1035 standards, while the subsequent patches expand OAuth configuration examples and reference credential helpers for Gmail and Outlook. The discussion includes minor formatting suggestions from Kristoffer Haugsbakk about commit message footnotes.

**macOS signal handling issue** — Torsten Bögershausen reports a compilation failure on older macOS systems related to `NOT_CONSTANT` and `sigfillset()`, proposing three potential solutions including a possible revert of commit 82e79c63642c. The issue stems from an interaction between Git's safety checks and macOS's header implementation.