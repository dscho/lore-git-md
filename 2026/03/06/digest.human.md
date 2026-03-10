# Git Mailing List Digest - 2026/03/06

**The day in brief.** A moderately busy day with 108 emails across 32 threads, featuring several significant developments. Key highlights include the finalization of configurable cover letter formatting, progress on the `git replay` revert feature, and a new URL-based configuration for partial clone filters. The day also saw multiple patch series nearing completion and several technical discussions about edge cases in existing functionality.

## Notable threads

### `git replay` revert capability matures

The long-running effort to add revert functionality to `git replay` reached a critical stage today, with Siddharth Asthana addressing final review feedback in preparation for v4. The series, primarily for GitLab's Gitaly service, now consolidates message formatting in `sequencer_format_revert_message()` per Phillip Wood's suggestion. Key technical decisions include processing reverts in chronological order (matching `git revert`) and dropping empty revert commits by default. While the core implementation appears sound, an emerging discussion about restructuring the interface as subcommands (`revert`, `pick`, `replay`) may lead to follow-up work.

### Configurable cover letter formatting finalized

Mirko Faina's series adding flexible cover letter formatting to `git format-patch` is now ready for merging after seven iterations. The implementation provides both command-line (`--cover-letter-format`) and configuration (`format.commitListFormat`) options, with new `%(count)` and `%(total)` placeholders for patch numbering. The design enforces a "log:" prefix requirement for custom formats to maintain consistency with existing behavior. Junio C Hamano provided final documentation tweaks, suggesting clearer wording about the relationship between CLI and config options.

### URL-based partial clone filters approved

Alan Braithwaite's feature to configure partial clone filters via URL patterns (`clone.<url>.defaultObjectFilter`) received maintainer approval after addressing all feedback. The implementation uses Git's urlmatch infrastructure with three specificity levels (domain/namespace/full path) and maintains safety boundaries by limiting to clone operations. A late objection from brian m. carlson about potential script disruptions was acknowledged but didn't block the feature, with Junio noting command-line overrides (`--filter`/`--no-filter`) provide sufficient control. The series now includes comprehensive test coverage and proper handling of the `--no-filter` override case.

### Rebase trailer support ready for merge

Phillip Wood and Li Chen's series adding `--trailer` support to `git rebase` is now in final form after eight iterations. The feature enables in-process trailer manipulation during rebase by refactoring trailer handling into reusable components. A final discussion about trailer ordering was resolved by maintaining consistency with `git commit` behavior (signoffs after other trailers). The implementation works across all rebase modes while preserving trailers through conflicts and properly handling fixup/squash operations. Junio engaged in final proofreading of the implementation details before merging.

### xdiff refactoring series revived

Junio noted that Ezekiel Newren's xdiff refactoring series (aimed at enabling future optimizations and Rust interoperability) can now be rebased after conflicts with Phillip Wood's merged cleanup patches. The 10-part series introduces an IVec type and restructures xdiff internals but faces unresolved concerns from Phillip about implementation safety and unclear Rust benefits. This maintainer nudge suggests the architectural work may still have value if the author addresses the outstanding feedback in a rebased version.

## In brief

**`the_repository` removal in wt-status.c** -- Shreyansh Paliwal's three-part series eliminating global state dependencies from `wt-status.c` has been fully merged, marking a significant milestone in Git's ongoing architectural cleanup.

**Repository statistics feature** -- Justin Tobler's series adding comprehensive object store metrics to `git repo` (tracking maximum object sizes, commit parent counts, and tree entry counts) received final ack from Junio and is queued for merging.

**Pre-add hook design reconsidered** -- Chandra Kethi-Reddy's `pre-add` hook series faces fundamental questions about its limited scope after Junio expressed doubts whether seeing only current `git add` changes provides sufficient value to justify inclusion.

**Branch name prefixing refined** -- Yoann Valeri's series evolved from a boolean config to a more flexible string-based approach for automatic branch naming, with Junio providing final guidance on proper option parsing structure to avoid needing a separate `--no-name-prefix` flag.

**MaintNotes typo fix** -- Silas Poulson's one-character documentation correction ("ttwo-dotted" to "two-dotted") was acknowledged by Junio and will be included in the next release notes.

**Fast-import re-signing feature** -- Justin Tobler's series adding a `re-sign-if-invalid` mode to `git fast-import` reached v2 with key specification support and proper handling of interoperability cases through explicit failure rather than silent conversion.

**Path.c cleanups merged** -- K Jayatheerth's refactoring series (removing unused includes, converting int to size_t for path lengths, and caching repository settings) was merged despite a redundant v3 submission from the author.

## On the radar

**Test failures on noexec filesystems** -- Ongoing investigation into credential helper and HTTP test failures when run from `/dev/shm` (mounted with noexec`) has identified the root cause but may prompt test infrastructure changes to handle such environments more gracefully.

**Worktree metadata design** -- Early discussion continues about adding descriptive metadata to worktrees, with Junio's suggestion to model it after `.git/description` files gaining support as a general solution rather than timestamp-specific approach.

**Stash pathspec documentation** -- Multiple contributors confirmed `git stash push` interprets pathspecs relative to current directory (not toplevel), prompting discussion about whether and how to document this behavior consistently across commands.