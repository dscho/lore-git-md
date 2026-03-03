# Git Mailing List Digest - 2025/06/10

**The day in brief.** A moderately busy day with 40 emails across 18 threads, featuring several notable developments: the completion of a `the_repository` removal series, an RFC for a new `git repo-info` command, and ongoing discussions about tag idempotency and pathspec behavior. The gitk external diff fix reached its final form after five iterations, while post-merge reviews identified subtle issues in the Perl removal series.

## Notable threads

### gitk external diff rename handling finalized

Tobias Boesch's fix for gitk's external diff functionality when handling renamed files reached its fifth and final iteration today. The solution has evolved from regex-based parsing to using Git's native rename detection via `git diff --find-renames`, addressing all technical and stylistic feedback from maintainer Johannes Sixt. The implementation now properly accounts for special commit ranges (nullid, cached states) while following Tcl conventions. This marks the successful conclusion of a focused improvement to gitk's Tcl script (42 lines added, 2 modified) that makes rename detection more robust when users scroll away from the initial diff position.

### Post-merge Perl removal review findings

SZEDER Gábor and Junio Hamano identified two issues in Patrick Steinhardt's already-merged Perl removal series (specifically patch 12/20). The more serious concern involves a pseudo-merge bitmap test where a broken sed conversion was passing due to insufficient output validation - the test succeeded because the conversion produced empty input rather than failing as it should. Junio reverted parts of the test back to Perl while adding the PERL_TEST_HELPERS prerequisite, acknowledging these edge cases still require Perl. This post-merge review highlights how test conversions can mask broken behavior when validation isn't thorough enough.

### Tag idempotency proposals diverge

The tag idempotency discussion saw competing proposals emerge today. Mirth Hickford suggested a middle ground for error messaging - warning only when tags would point to different commits - while Junio Hamano took a different approach by implementing partial idempotency directly in `builtin/tag.c`. Junio's version allows repeated lightweight tag creation when pointing to the same commit (still requiring `--force` for annotated tags), introducing an asymmetry between tag types. The thread shows the community grappling with how to balance safety and convenience in tag operations, with no clear consensus yet on the right technical or UX approach.

### `git repo-info` RFC sparks format debate

Lucas Seiki Oshiro proposed a new `git repo-info` command as a GSoC project, aiming to split repository metadata functionality from `git rev-parse`. The 5-patch series implements JSON and plaintext output for reference storage format, bare status, and shallow detection. While Kristoffer Haugsbakk endorsed the concept, Junio Hamano raised concerns about the plaintext format's robustness with pathnames containing newlines, suggesting JSON as the initial baseline. The discussion reveals tension between human-readable output and unambiguous parsing, with the JSON schema likely to be finalized first.

### macOS test prerequisites verified

D. Ben Knoble completed comprehensive testing of Ramsay Jones's fix for missing test prerequisites (POSIXPERM, BSLASHPSPEC, EXECKEEPSPID) on Darwin/macOS systems. After running all 31,723 tests successfully on actual macOS hardware, Knoble provided an Acked-by tag, confirming the patch restores correct test behavior without regressions. This concludes the verification process for a fix that stemmed from lack of macOS testing access in the initial submission.

## In brief

**gitk external diff rename handling** -- Tobias Boesch's five-iteration fix for gitk's rename detection when using external diffs is finalized, replacing regex parsing with native Git rename detection.

**Perl removal test issues** -- Post-merge review finds two edge cases in test conversions from Perl to shell/awk/sed, with one test passing incorrectly due to broken sed syntax producing empty input.

**Memory leak in boundary traversal** -- Final fix addresses a lingering edge case where `roots_bitmap` would leak during successful pseudo-merge boundary traversal in pack-bitmap.c.

**rebase trailer infrastructure** -- New `amend_strbuf_with_trailers()` function moves trailer handling in-process for a proposed `--trailer` rebase option, eliminating fork/exec overhead.

**`the_repository` removal in preload-index** -- Ayush Chandekar's GSoC series eliminating `core_preload_index` and `the_repository` from preload-index.c gets maintainer approval.

**`git branch --force` multi-branch proposal** -- Andrea Stacchiotti suggests `cp`-like semantics for multiple branch operations, sparking discussion about configuration inheritance and force semantics.

**Editor template whitespace behaviors** -- Follow-ups document that `git commit -v` introduces trailing whitespace on blank lines, similar to the recently fixed notes template issue.

## On the radar

**Pathspec exclude pattern behavior** -- The thread about whether unmatched exclude patterns should error appears to be converging on maintaining silent behavior, though the original empty-repo edge case remains unresolved.

**Interactive rebase error handling** -- Phillip Wood suggests improving `get_commit_format()` to avoid direct `die()` calls during invalid instruction handling, which would enable proper cleanup of stashed changes.