# Git Mailing List Digest — 2026/02/01

**The day in brief.** A moderately active day with 15 emails across 8 threads, dominated by documentation standardization work and a few bugfixes. The most notable developments include the conclusion of a discussion about merge commit warnings in `format-patch`, a quick fix for `git blame` coloring with ignored revisions, and continued progress on Jean-Noël Avila's man page standardization effort. Newcomer engagement for GSoC 2026 also begins in earnest.

## Notable threads

**Merge commit warning proposal withdrawn** — Dominique Martinet re-evaluated the usefulness of adding warnings when `git format-patch` skips merge commits, concluding after real-world testing that the warning would provide minimal benefit while potentially interfering with existing test expectations. The thread, which began as an RFC patch series on January 3, appears to be concluding with consensus that the change isn't worth pursuing given the complexity of specifying merge-only ranges and the likelihood that users would encounter the scenario unintentionally.

**Documentation standardization advances** — Jean-Noël Avila and Kristoffer Haugsbakk continued their meticulous collaboration on converting Git's man pages to synopsis-style AsciiDoc format. Today's exchanges focused on `git-submodule`, `git-clone`, and `git-show` documentation, with detailed discussions about option formatting consistency (particularly the "stuck" vs "unstuck" forms of long options) and rendering issues with special characters in decoration options. The work remains strictly presentational, with no behavior changes, but demonstrates the project's exacting standards for documentation quality.

**Blame coloring fix for ignored revisions** — René Scharfe quickly addressed a bug where `git blame --color-lines` didn't properly color adjacent lines that blame the same commit after applying `--ignore-rev`. The fix modifies the coloring logic in `builtin/blame.c` to check the previous line's commit when determining whether to apply the repeated-line color. A new test case verifies the corrected behavior with a whitespace-changing commit that should show uniform coloring when ignored.

## In brief

**Newcomer engagement for GSoC 2026** — Soutrik Das introduced themselves as a prospective Google Summer of Code participant, beginning the standard onboarding process with guidance from Deveshi Dwivedi about microprojects and initial contributions.

**CONTRIBUTING.md link fix** — A documentation patch updates a broken relative link to use an absolute git-scm.com URL, maintaining consistency with other links in the file while fixing GitHub's rendering issue.

**Clone protocol v0/v1 segfault fix** — A minimal change prevents a segmentation fault when cloning with `--revision` using older protocols, adding a NULL check for `remote_head_points_at->peer_ref`.

**Submodule traversal documentation** — Jean-Noël Avila provided review feedback on the final documentation patch for new `--reverse-traversal` flags, ensuring alignment with the synopsis-style markup conventions.