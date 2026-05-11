# Git Mailing List Digest - 2026/05/10 (Sunday)

**The day in brief.** A moderately busy Sunday with 31 emails across 13 threads, featuring final polish for several long-running discussions. Key developments include resolution of the `--max-count-oldest` feature, Windows large-object handling reaching consensus, and substantive documentation improvements for `git log --follow`. The day saw productive technical discussions with multiple patches nearing merge readiness.

## Notable threads

### `--max-count-oldest` feature finalized

The `--max-count-oldest` feature (showing oldest commits in revision walks) has reached its final form after multiple iterations. Mirko Faina addressed the last remaining issue - a test portability problem with GNU `tac` - by replacing it with a portable `--reverse | head` combination. Junio C Hamano confirmed this resolves the final blocker, with all substantive technical, interface, and documentation concerns already addressed in prior rounds. The feature preserves its optimized O(K) space and O(N) time characteristics while properly integrating with Git's revision walker architecture.

### Windows large-object handling consensus

Johannes Schindelin's series handling large objects (>4GB) on Windows has cleared its last technical hurdle. Torsten Bögershausen retracted his earlier concern about memory alignment when reading varint-encoded object sizes, confirming Junio Hamano's explanation that the code safely handles byte-by-byte reading without unaligned access risks. Junio also modified the CI strategy to run expensive tests on both PR builds and integration branch pushes, ensuring earlier detection of issues. The series now appears ready for integration with all technical concerns resolved.

### Maintenance daemonization locking fix debate

Taylor Blau and Derrick Stolee engaged in substantive discussion about fixing repository corruption risks in `git maintenance run --detach`. Blau proposed a comprehensive solution with `reassign_tempfile_ownership()` to properly transfer locks during fork(), while Stolee leaned toward reverting the geometric repack default for v2.54.1 as a safer short-term fix. Blau provided a FIFO-based test demonstrating proper lock cleanup, though portability concerns remain. The thread awaits Stolee's evaluation of whether the test adequately addresses reliability concerns.

### `git log --follow` documentation refined

Tamir Duberstein's documentation patch for `git log --follow` and `--no-follow` reached its final form through rigorous review. Junio Hamano provided authoritative clarification that the feature is explicitly designed to work only with single filenames in linear history, pushing back on wording that implied partial functionality for other cases. The documentation now precisely delineates between defined behavior (single file, linear history) and undefined cases (directories, non-linear history), reflecting Git's philosophy of accurate rather than aspirational documentation.

## In brief

**Submodule fetchJobs configuration fix** -- Saagar Jha's v4 patch corrects pointer indirection in submodule configuration reading, ensuring `.gitmodules` settings for parallel fetches are respected. Junio Hamano suggested test style improvements which were incorporated.

**Terminal output corruption fix** -- René Scharfe's patch moving ANSI clear sequences to prefix position fixes character truncation in full-width terminal lines. Junio Hamano endorsed the approach as conceptually simpler and more predictable.

**Negative diff context validation** -- Michael Montalbo's series converting diff context options to unsigned values with validation is ready for merging after final documentation clarifications about `PARSE_OPT_NONEG`.

**Rebase --update-refs edge case** -- A bugfix ensuring rebase only processes branch refs when `rebase.instructionFormat` includes `%d` received Phillip Wood's Reviewed-by and Junio Hamano's confirmation of correctness.

**Nedmalloc removal complete** -- Johannes Schindelin's series removing the unmaintained nedmalloc allocator from Windows builds has been merged, simplifying the codebase by deleting over 5,700 lines.

**git-svn duplicate option warning** -- A bug report identifies a warning in `git-svn` due to duplicate option specifications that will become fatal in future Getopt::Long releases. The straightforward fix would remove one of the duplicate mappings.

## On the radar

**Branch pruning automation** -- Harald Nordgren's proposal for composable branch pruning commands remains at a design crossroads after Johannes Sixt's architectural concerns about fetch integration. The thread was briefly sidetracked by an email misdirection but the core discussion awaits resolution.