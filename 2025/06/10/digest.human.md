# Git Mailing List Digest - 2025/06/10

**The day in brief.** A moderately busy day with 40 emails across 18 threads, featuring several notable developments: the conclusion of a long-running gitk external diff bug, post-merge discoveries in the Perl removal series, and the introduction of a new `git repo-info` command proposal. The most significant technical exchanges centered around tag idempotency semantics and pathspec exclude pattern behavior.

## Notable threads

### gitk external diff rename handling finalized

Tobias Boesch's five-iteration effort to fix gitk's external diff functionality for renamed files reached completion today. The final version replaces regex-based filename parsing with direct Git command execution (`git diff --find-renames`), addressing all of Johannes Sixt's technical and style concerns. The solution now properly accounts for special commit ranges (nullid, cached states) while maintaining Tcl style conventions. This marks the resolution of a long-standing usability issue where gitk's external diff feature would fail when handling renamed files if users scrolled away from the initial diff position.

### Post-merge Perl removal issues surface

SZEDER Gábor identified two issues in Patrick Steinhardt's already-merged Perl removal series (specifically patch 12/20 converting test helpers). The more serious concern involves a syntactically incorrect sed command in t5333-pseudo-merge-bitmaps.sh that was passing tests despite being broken - revealing a test coverage gap where silent failures could go undetected. Junio Hamano noted the test was passing trivially on empty input rather than properly validating the conversion. A follow-up patch reverted the problematic conversions back to Perl while adding the PERL_TEST_HELPERS prerequisite, confirming these edge cases still require Perl.

### Tag idempotency debate evolves

The discussion around tag creation idempotency took an unexpected turn as Junio Hamano proposed a technical implementation diverging from the ongoing message-only improvements. His patch makes lightweight tag creation idempotent when pointing to the same commit, while maintaining the existing behavior for annotated tags. This contrasts with Mirth Hickford's earlier compromise proposal to improve error messages without changing behavior. The thread now encompasses both philosophical (should tags be idempotent?) and technical (how to handle configuration inheritance?) dimensions, with no clear consensus yet emerging.

### Pathspec exclude pattern consensus forms

After extensive discussion, maintainers appear to be converging on preserving the historical behavior where exclude patterns (unlike positive pathspecs) don't error on non-matches. Piotr Siupa reinforced Junio Hamano's position with practical scripting examples, noting that erroring on unmatched excludes would break workflows like `git add -- \*.c ':!auto-generated.c'`. While the original empty-repository edge case remains technically open, the broader semantic debate seems settled in favor of silent non-matches for excludes, aligning with .gitignore behavior.

### New `git repo-info` command proposed

Lucas Seiki Oshiro introduced an RFC for a new `git repo-info` command as part of a GSoC project, aiming to split repository metadata functionality from the overloaded `git rev-parse`. The 5-patch series implements JSON and plaintext output for reference storage format, bare repository status, and shallow detection. Junio Hamano provided feedback favoring JSON as the initial format due to its robustness with arbitrary data, while cautioning about newline handling in the proposed plaintext version. The proposal represents a thoughtful approach to command specialization, following Git's pattern of splitting complex commands like checkout into focused tools.

## In brief

**Memory leak fixes** -- Lidong Yan's v3 patch fixing a revision walking leak during merge conflicts is now finalized with corrected test formatting, while a separate boundary traversal leak in pack-bitmap.c was caught and fixed post-merge.

**macOS test prerequisites** -- D. Ben Knoble completed verification of Ramsay Jones's fix for missing test prerequisites on Darwin systems, confirming all 31,723 tests pass with the corrected POSIXPERM, BSLASHPSPEC, and EXECKEEPSPID handling.

**rebase trailer infrastructure** -- A 2-patch series introduced in-process trailer handling for `git rebase --trailer`, eliminating fork/exec overhead while maintaining all edge case behaviors of the external `interpret-trailers` helper.

**`the_repository` removal progress** -- Ayush Chandekar's GSoC contribution to remove `the_repository` from preload-index.c received maintainer approval, localizing config access and using the index_state's repo pointer instead of globals.

**NonStop RC2 verification** -- Randall Becker reported successful testing of Git v2.50.0-rc2 on the NonStop platform without TCL support, contributing to release confidence for this less common configuration.

## On the radar

**`git branch` multi-target operations** -- Andrea Stacchiotti's proposal to add `cp`-like semantics for multiple branch operations sparked design discussions about configuration inheritance and whether `--force` is the right conceptual model, with Junio suggesting the feature may need refinement before proceeding.

**Interactive rebase error handling** -- Phillip Wood suggested deeper changes to `get_commit_format()`'s error handling to properly clean up stashed changes during interactive rebase, expanding the scope from Øystein Walle's localized fix.