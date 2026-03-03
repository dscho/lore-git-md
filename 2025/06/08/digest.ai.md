# Git Mailing List Digest - 2025/06/08

**The day in brief.** A moderately busy Sunday with 49 emails across 11 threads, featuring several patch series reaching maturity. Key developments include final approvals for the `imap-send` overhaul and `git stash` submodule fixes, plus new documentation work and a `the_repository` removal series. Junio Hamano also contributed a new `--compact-summary` option for merges.

## Notable threads

### `imap-send` overhaul reaches final approval

The 15th iteration of Aditya Garg's comprehensive `imap-send` improvements received maintainer approval after addressing all technical concerns. This series fixes a critical configuration parsing regression (broken since Git 2.46.0) and adds modern authentication methods including OAuth2.0 (both standard OAUTHBEARER and Google's XOAUTH2) and RFC-compliant PLAIN authentication. The final version reorganizes patches per Junio's request, moving CRAM-MD5 handling earlier in the series. New features include folder management via `--list` and `--folder` options, plus user experience improvements like better credential prompts. With all foundational changes approved, only minor polish patches remain for review.

### `git stash` submodule branch name fix finalized

K Jayatheerth's solution for preserving superproject branch names in stash messages when working with submodules reached its final form after thorough review. The patch addresses buffer reuse in `refs_resolve_ref_unsafe()` that caused submodule branch names to incorrectly appear in superproject stash entries. The implementation uses `xstrdup()` with proper memory management, now incorporating all feedback from Junio Hamano and René Scharfe. Junio's final review suggested minor optional style tweaks (test naming, whitespace cleanup) but confirmed the technical approach is sound. This concludes a focused thread that identified and fixed a subtle but important edge case in submodule workflows.

### `the_repository` removal advances in prune builtin

Ayush Chandekar contributed a focused 2-patch series removing `the_repository` usage from the prune builtin as part of GSoC work. The first patch migrates the `repository_format_precious_objects` global into `struct repository`, while the second updates `builtin/prune.c` to use the passed `repo` parameter instead of the global. These mechanical changes follow established patterns in the ongoing effort to eliminate global state. Christian Couder provided patch submission advice (version descriptions, reviewer credits) but no technical concerns, suggesting this straightforward series is ready for merging.

### New `--compact-summary` option for merges

Junio Hamano proposed a new merge output format with `--compact-summary`, building on diffstat to mark created/deleted files with `(new)`/`(gone)` annotations. The implementation adds a `MERGE_SHOW_COMPACTSUMMARY` flag and integrates with existing diff machinery, appearing in both `merge` and `pull` commands. The change is well-scoped with comprehensive tests and documentation, offering more context than standard diffstat without significantly increasing output size. As a maintainer-authored patch targeting a clear user experience improvement, this will likely merge quickly.

## In brief

**Solaris build updates** -- Brad Smith updated Solaris build rules to better reflect function availability across versions, with Collin Funk verifying the changes work on Solaris 10 and 11.3.

**Submodule config management** -- K Jayatheerth's v8 series now handles redundant `.active` entries by checking path patterns, completing discussion with Junio about implementation scope.

**Memory leak fix in submodule update** -- A straightforward patch plugs a `remote_name` leak when `remote_submodule_branch()` fails during submodule update.

**Bash completion customization** -- A new `completion.forRefSort` option lets users customize ref completion order, defaulting to alphabetical but allowing sorts like `-committerdate`.

**Documentation conversion** -- Jean-Noël Avila began converting the massive `git-log` man page to synopsis format, with 9 patches standardizing syntax across related docs.

## On the radar

**Workflow optimization discussion** -- A user question about streamlining amend-then-fixup workflows prompted comprehensive answers about `--amend`, `--fixup`, and Git 2.44.0's improved `--autosquash` behavior, potentially inspiring future documentation improvements.