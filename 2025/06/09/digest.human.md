# Git Mailing List Digest - 2025/06/09

## The day in brief

A busy Monday with 76 emails across 17 threads, dominated by the finalization of a major `imap-send` overhaul and discussions around tag creation behavior. The release of Git v2.50.0-rc2 brought notable performance improvements and new features, while several bugfixes addressed memory leaks and edge cases in rebase and revision walking.

## Notable threads

### `imap-send` resurrection complete

After 18 iterations, Aditya Garg's comprehensive `imap-send` series is ready for merging. The v18 patches fix a critical configuration parsing regression (broken since Git 2.46.0), add OAuth2.0 authentication support (both RFC-standard OAUTHBEARER and Google's XOAUTH2), implement proper PLAIN authentication for OpenSSL, and introduce new folder management features (`--folder` override and `--list` enumeration). The series also includes numerous memory leak fixes and user experience improvements like better credential prompts. Junio Hamano has approved the core changes, with only minor documentation and style nits addressed in the final version.

### Tag creation behavior debate

A lively discussion emerged around improving tag conflict handling, with three distinct proposals:
1. Showing more detailed error messages when tags exist (original suggestion)
2. Keeping the current simple "tag exists" message (Junio's preference)
3. Making tag creation idempotent when commits match (Hilco Wijbenga's proposal)

The thread surfaced important considerations about tag type differences (lightweight vs annotated) and the principle of least surprise. While no consensus was reached, the discussion clarified technical constraints any solution would need to address, particularly around preserving tag metadata in idempotent operations.

### Git v2.50.0-rc2 released

Junio announced the second release candidate for Git 2.50, featuring 604 non-merge commits from 85 contributors. Highlights include:
- New `--combine-cruft-below-size` for `git repack`
- TCP keepalive configuration for HTTP
- `git reflog drop` subcommand
- Machine-parsable `rev-list` output
- Performance improvements in ref handling and reftable operations

The release continues Git's modernization efforts with MIDX bitmaps, reftable optimizations, and reduced `the_repository` usage, alongside numerous Windows/ARM64 fixes and test improvements.

### Interactive rebase state cleanup

Øystein Walle fixed an edge case where invalid `rebase.instructionFormat` configurations could leave repositories in a half-initialized state. The patch reorders operations in `do_interactive_rebase()` to validate the instruction format before setting up rebase metadata, preventing inconsistent states when format parsing fails. Junio accepted the solution with minor style suggestions.

## In brief

**Memory leak in boundary traversal** -- Taylor Blau fixes a remaining edge case where `roots_bitmap` could leak during pseudo-merge cascading, adding a test case to verify the fix.

**Revision walking leak fixed** -- Lidong Yan's v2 patch finalizes the solution for a `prune` array leak in `prepare_show_merge()`, now with comprehensive test coverage for merge conflict scenarios.

**Documentation formatting fix** -- Kristoffer Haugsbakk corrects a malformed `linkit:` reference in the maintenance docs, changing it to the proper `linkgit:` syntax for cross-references.

**macOS test prerequisites** -- Ramsay Jones adds missing test prerequisites (POSIXPERM, BSLASHPSPEC, EXECKEEPSPID) for Darwin systems, fixing incorrect test skipping behavior.

**Notes editor whitespace conclusion** -- Threads about removing trailing whitespace from notes editor templates reached consensus on keeping the focused `notes.c` solution rather than expanding to `pretty.c`.

## On the radar

**Force push confirmation debate** -- Aditya Garg's proposal for interactive `git push -f` confirmations met resistance from Junio, who argued safety mechanisms should be more general than command-specific prompts. The discussion appears resolved with no changes planned.

**Windows build fixes pending** -- Sebastian Siewior is preparing fixes for Windows compilation failures in the `compat/bswap.h` refactoring, with both a regression revert and corrected cleanup patch expected soon.