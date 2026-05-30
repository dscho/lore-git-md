Here's the daily digest for May 29, 2026:

## The day in brief

A moderately busy Friday with 46 emails across 16 threads, featuring performance optimizations, documentation refinements, and workflow discussions. The standout items include Taylor Blau's bitmap optimization series reaching final approval and Michael Montalbo's RFC for external diff drivers advancing to v3 with comprehensive blame integration.

## Notable threads

### Bitmap optimization series concludes

Jeff King (Peff) gave final approval to Taylor Blau's eight-patch series optimizing pack-bitmap-write performance after thorough review. The changes achieve 60% faster generation times and 72% smaller bitmaps for large repositories while maintaining backward compatibility. The discussion included detailed performance analysis showing path-walk integration can reduce server-side operations by 98.8% for recent fetches when properly configured. With all technical concerns addressed, this major optimization effort is now ready for integration.

### External diff driver RFC advances

Michael Montalbo's RFC for `diff.<driver>.process` reached v3 with blame integration now implemented. The series allows external tools to inject diff hunks into Git's pipelines while preserving features like word-diff and color-moved. New in this version are subprocess management refactoring, improved error handling, SIGPIPE protection, and comprehensive test coverage (708 lines). The blame integration demonstrates skipping reformat-only commits via the protocol's "no hunks" response. While performance considerations remain (~17% blame overhead), the implementation appears technically sound with all major components now in place.

### HTTP pack index leak fix finalized

Jeff King (Peff) completed his review of Lorenzo Pegorari's fix for memory leaks in HTTP pack index error paths. The discussion explored tempfile cleanup behavior, ultimately confirming that removing a redundant `unlink()` call is safe since tempfile registration ensures cleanup at process exit. Peff considered but rejected a more complex refactoring as unnecessary for this case. The exchange exemplifies Git's review culture of thoroughly evaluating alternatives while maintaining pragmatism about when to stop optimizing.

### `git son` command vs submodule enhancements

The proposed `git son` command for creating loosely-coupled child repositories faced a substantive counter-proposal from Claus Schneider, who suggested enhancing submodules instead. Schneider's pull request (#1987) modifies submodule behavior to better support branch-tracking and ignore patterns, potentially addressing the original use case without new porcelain. This shifts the discussion from implementation details to fundamental design questions about whether the functionality belongs in a new command or existing infrastructure.

### Index-pack delta resolution optimization

A focused one-line change to index-pack's delta resolution showed significant performance gains. By removing a premature `free_base_data()` call, the patch leverages the delta base cache more effectively, reducing wall time by 15-16% on linux.git with minimal memory overhead (1.9% RSS increase). The well-documented change demonstrates how small but targeted optimizations can yield substantial improvements in core operations.

## In brief

**Line-log integration merged** -- Michael Montalbo's series unifying `git log -L` with the standard diff pipeline reached `next` after D. Ben Knoble confirmed the test syntax changes look sound.

**Deprecated commit list functions removed** -- Kristoffer Haugsbakk's cleanup removing long-deprecated functions from commit.h sparked discussion about Git's deprecation workflow efficiency, though the technical changes were uncontroversial.

**Redundant sort removed from commit-reach** -- René Scharfe eliminated an unnecessary date-based sort in `paint_down_to_common()`, with Peff suggesting potential follow-up optimizations using NULL result pointers.

**macOS linker warning resolved** -- Harald Nordgren's v2 patch addresses Xcode 16.3+ warnings by conditionally adding `-fno-common` to BASIC_CFLAGS when needed.

**Windows pathspec test fix** -- Kristofer Karlsson's workaround for backslash pattern handling in `git ls-files` tests was queued, though deeper pathspec parsing issues remain.

**Merge documentation update** -- A patch adding `--message` to `git-merge` docs revealed broader gaps in `[no-]` variant documentation that may need addressing.

**Pre-push hook argument handling** -- Wesley Schwengle's report about unexpected arguments in config-based hooks led to discussion of workarounds and documentation improvements, with proper quoting (`"npm run test #"`) confirmed as a solution.

**Ancient timestamp documentation** -- Junio confirmed the `@` prefix syntax for very old dates (added in 2012 but undocumented) should be documented in date-formats.adoc, inviting Luna Schwalbe to submit a patch.

**Typo fixes** -- Weijie Yuan submitted mechanical corrections for 14 documentation and code comment issues across multiple files.

## On the radar

**Selective fetch strategies** -- Matthew Hughes' workflow question about managing fetches in large repositories generated three distinct approaches (selective refspecs, fetch-all with maintenance, and custom scripts), highlighting an area where core Git might benefit from enhanced functionality.