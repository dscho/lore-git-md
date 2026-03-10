# Git Mailing List Digest — 2026/02/01

**The day in brief.** A moderately active day with 15 emails across 8 threads, dominated by documentation standardization work and a few bugfixes. The most notable developments include a withdrawn RFC about merge commit warnings in `format-patch`, a fix for `git blame` coloring with ignored revisions, and continued progress on Jean-Noël Avila's man page standardization effort.

## Notable threads

### Documentation standardization advances

Jean-Noël Avila's multi-year effort to standardize Git's man pages to AsciiDoc synopsis style saw extensive review activity, with Kristoffer Haugsbakk providing detailed feedback on several documentation conversions. The discussion focused on:

- Consistent formatting of command-line options (stuck vs. unstuck forms)
- Proper use of AsciiDoc macros for notes and warnings
- Cross-toolchain compatibility between asciidoc.py and asciidoctor
- Rendering issues with special characters in decoration options

The meticulous review process demonstrates the project's commitment to documentation quality, with even minor formatting inconsistencies being systematically addressed. Avila acknowledged the feedback and plans to incorporate the suggested improvements in future iterations.

### `git blame` coloring fix with ignored revisions

René Scharfe submitted a targeted fix for an interaction bug between `--color-lines` and `--ignore-rev` in `git blame`. The issue caused incorrect coloring when multiple consecutive lines were assigned to the same commit after ignoring a revision. The fix modifies the line coloring logic to properly account for the post-ignored revision state.

The patch includes a new test case that verifies the corrected behavior by creating a commit that changes whitespace and checking the coloring when that revision is ignored. This small but important fix addresses a genuine bug in how two blame features interact.

### Merge commit warning proposal withdrawn

Dominique Martinet reconsidered an RFC proposal to add warnings about skipped merge commits in `git format-patch`. After encountering a real-world case and deeper consideration of edge cases, Dominique concluded the warning would provide limited practical benefit while potentially interfering with patch counting in tests.

The thread appears to be concluding with consensus that the change isn't worth pursuing, demonstrating how real-world usage can reshape initial proposals during Git's review process.

## In brief

**Clone command fix** -- A segmentation fault when using `--revision` with older protocols (v0/v1) is fixed by adding a NULL check for `peer_ref`.

**CONTRIBUTING.md link update** -- A broken relative link is updated to use an absolute git-scm.com URL, maintaining consistency with other documentation references.

**Submodule documentation review** -- Jean-Noël Avila provides style feedback on new `--reverse-traversal` flag documentation to align with the synopsis standardization effort.

**New contributor engagement** -- A prospective Google Summer of Code participant begins exploring contribution pathways, receiving guidance about microprojects and the patch submission process.