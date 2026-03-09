# Git Mailing List Digest - 2025/06/09 -- 2025/06/15

## The week in brief

A busy week with 327 emails across 108 threads saw the Git project make significant progress on multiple fronts. The release of Git v2.50.0-rc2 brought notable performance improvements and new features, while major discussions centered around tag creation behavior, `imap-send` modernization, and repository metadata handling. The week was marked by both technical refinements (like the completion of Aditya Garg's 18-iteration `imap-send` series) and philosophical debates (notably around memory management practices and command consolidation). Platform compatibility work continued with Solaris and FreeBSD fixes, while the long-running `the_repository` removal effort saw steady progress.

## Key developments

### `imap-send` modernization completes

After 18 iterations spanning several months, Aditya Garg's comprehensive overhaul of `git imap-send` is ready for merging. The final version (v18) fixes a configuration parsing regression dating back to Git 2.46.0, adds OAuth2.0 authentication support (both standard OAUTHBEARER and Google's XOAUTH2), implements proper PLAIN authentication for OpenSSL, and introduces new folder management features including `--folder` override and `--list` enumeration. The series also includes numerous memory leak fixes and user experience improvements like better credential prompts. Junio Hamano approved the core changes after addressing minor documentation and style nits, marking the conclusion of one of the longest-running patch series in recent memory.

### Tag creation behavior evolves

A lively multi-day discussion explored improving tag conflict handling, with three competing proposals:
1. Showing more detailed error messages when tags exist (original suggestion)
2. Keeping the current simple "tag exists" message (Junio's initial position)
3. Making tag creation idempotent when commits match (Hilco Wijbenga's proposal)

The debate surfaced important technical constraints around tag type differences (lightweight vs annotated) and metadata preservation. Junio ultimately proposed a technical implementation making lightweight tag creation idempotent when pointing to the same commit, while maintaining existing behavior for annotated tags. This compromise solution addresses the usability concern while respecting Git's historical semantics, though some philosophical questions about tag idempotency remain open.

### Git v2.50.0-rc2 released

Junio announced the second release candidate for Git 2.50, featuring 604 non-merge commits from 85 contributors. Highlights include:
- New `--combine-cruft-below-size` for `git repack`
- TCP keepalive configuration for HTTP
- `git reflog drop` subcommand
- Machine-parsable `rev-list` output
- Performance improvements in ref handling and reftable operations

The release continues Git's modernization efforts with MIDX bitmaps, reftable optimizations, and reduced `the_repository` usage, alongside numerous Windows/ARM64 fixes and test improvements. Platform-specific updates like Git for Windows 2.50.0-rc2 (with ReFS drive fixes) and successful NonStop verification indicate broad compatibility.

### Promisor-remote protocol validation finalized

Christian Couder's 5-patch series extending the promisor-remote protocol to support configurable validation of remote attributes reached its final form (v4). The implementation allows servers to advertise additional fields like `partialCloneFilter` and `token` via `promisor.sendFields`, while clients can validate these using `promisor.checkFields`. Key changes in v4 include replacing `struct strvec` with `struct promisor_info` per Junio's suggestion, improved documentation formatting, and standardized terminology. The series addresses all feedback from Patrick Steinhardt, Karthik Nayak, and Junio Hamano, with passing CI tests indicating readiness for merging.

### Stash import/export feature lands

brian m. carlson's eighth and final version of the stash import/export series introduces new `git stash export` and `import` subcommands enabling bidirectional transfer of stashes between repositories. The implementation stores commit chains under `refs/stash-export/` and includes 268 lines of test coverage. Phillip Wood's review feedback was addressed regarding ownership semantics in `write_commit_with_parents()` and invalid argument combinations. This long-running work provides a robust solution for sharing stash states across repository boundaries, completing a feature request that has existed since the early days of Git's stash functionality.

## In brief

**Memory leak fixes** -- Multiple leaks were addressed including Taylor Blau's boundary traversal fix in pseudo-merge cascading, Lidong Yan's revision walking leak during merge conflicts, and Jinyao Guo's mailinfo boundary handling.

**Platform compatibility** -- Solaris sed `-E` flag incompatibility was resolved with portable rewrites, while FreeBSD 6.0 became the new minimum supported version with legacy build settings removed.

**Documentation improvements** -- Kristoffer Haugsbakk corrected malformed `linkgit:` references while Jean-Noël Avila's synopsis-style conversion continued with several documentation patches.

**Test infrastructure** -- Ramsay Jones and D. Ben Knoble fixed macOS test prerequisites (POSIXPERM, BSLASHPSPEC, EXECKEEPSPID) ensuring proper test execution on Darwin systems.

**`the_repository` removal** -- Steady progress continued with patches from Lidong Yan (run_builtin conversion) and Ayush Chandekar (preload-index.c changes), alongside design discussions about configuration access patterns.

**Batch reference updates** -- Patrick Steinhardt's fixes for batched reference updates reached final approval, addressing directory/file conflict handling and failed update scenarios in `git receive-pack`.

**Compact merge summaries** -- Junio introduced a new `--compact-summary` option for merge/pull operations that modifies diffstat output to explicitly mark created/deleted paths.

**New command proposals** -- Lucas Seiki Oshiro's `git repo-info` (extracting metadata from rev-parse) and a GSoC student's `git refs list` (consolidating ref listing) sparked interface design discussions.

**Localization updates** -- Jiang Xin submitted translation updates including a new Irish (ga.po) translation, while Alexander Shopov updated Bulgarian git-gui localization.

## Looking ahead

**Git 2.50.0 final release** -- With rc2 testing well and only minor documentation fixes outstanding, the final release appears imminent barring last-minute issues.

**Batch operations debate** -- The architectural discussion about implementing batch operations in `git branch` vs `update-ref` continues, with interface design questions still open.

**Rustification effort** -- While not active this week, Ezekiel Newren's work to introduce Rust code remains a significant ongoing effort likely to resurface.

**Git Merge 2025** -- Taylor Blau's announcement of September's conference schedule may prompt increased activity around planned features and demonstrations.