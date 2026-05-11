# Git Mailing List Digest - 2026/05/10 (Sunday)

**The day in brief.** A moderately busy Sunday with 31 emails across 13 threads, featuring final polish on several long-running topics. Key developments include resolution of the Windows large-object handling series, final tweaks to the `--max-count-oldest` feature, and substantive documentation improvements for `--follow`/`--no-follow`. The day also saw continued discussion about maintenance daemonization safety and a new `git-svn` Perl warning.

## Notable threads

### Windows large-object handling finalized

Johannes Schindelin's series addressing Windows platform issues with objects >4GB reached resolution today. The final technical concern - about potential memory alignment issues when reading varint-encoded object sizes - was resolved when Torsten Bögershausen acknowledged his earlier concern was mistaken after Junio Hamano's explanation. Junio also contributed a CI configuration change to run expensive tests on PR builds and integration branches, ensuring earlier detection of issues. With all technical questions settled, this series is now ready for integration, marking an important step in Windows platform support.

### Maintenance daemonization safety debate

The thread about repository corruption risks in `git maintenance run --detach` saw substantive discussion today between Taylor Blau and Derrick Stolee. Stolee raised concerns about the scope of Taylor's proposed `reassign_tempfile_ownership()` solution, suggesting a revert of the geometric repack default for v2.54.1 as a safer short-term fix. Taylor responded with POSIX documentation confirming atexit() behavior and provided a FIFO-based test demonstrating proper lock cleanup. While leaning toward Stolee's revert suggestion for stability, Taylor's technical responses strengthened the case for the comprehensive solution. The thread now awaits Stolee's evaluation of whether the test adequately addresses reliability concerns.

### `--follow`/`--no-follow` documentation finalized

Tamir Duberstein's documentation patch series for `git log --follow` and `--no-follow` reached its final form today with Junio Hamano's authoritative clarification about the feature's limitations. Junio emphasized that `--follow` was explicitly designed only for single filenames in linear history, pushing back on wording that implied partial functionality for other cases. The v3 patch now precisely documents these constraints, including undefined behavior with directories and non-linear history. This exemplifies Git's documentation philosophy of describing actual behavior rather than aspirational features, even for long-standing options.

## In brief

**`--max-count-oldest` final polish** -- Mirko Faina addresses the last test portability issue in this feature, replacing GNU `tac` with a portable `--reverse | head` combination. The change doesn't affect functionality but ensures the test suite works across platforms.

**Submodule fetchJobs bugfix** -- Junio Hamano provides technical context about why the pointer indirection bug went undetected, noting the `void*` parameter type masked the issue. The v4 patch now includes his suggested test style improvements using `git -C` and `test_grep`.

**Terminal output corruption fix** -- René Scharfe's patch moving the ANSI clear sequence to fix character truncation in full-width terminal lines receives maintainer approval from Junio Hamano, who notes the prefix approach provides more predictable behavior.

**Negative diff context values** -- Michael Montalbo's series fixing negative value handling for `-U`/`--unified` and `--inter-hunk-context` reaches its final form with documentation adjustments per Junio's feedback about `PARSE_OPT_NONEG` scope.

**Rebase --update-refs edge case** -- Phillip Wood confirms the current patch correctly handles branch refs through existing filtering in `sequencer.c`, while identifying a related issue in `make_script_with_merges()` that needs similar treatment.

**Nedmalloc removal complete** -- Junio Hamano merges Johannes Schindelin's series removing the unmaintained nedmalloc allocator from Windows builds, closing a chapter in Git's history by deleting over 5,700 lines of obsolete code.

**Documentation typo fixes** -- Junio provides feedback on Andrew Kreimer's patch fixing typos across multiple files, suggesting it be split into logical groupings and raising questions about unmaintained translation files.

## On the radar

**`git-svn` Perl warning** -- A new bug report highlights a warning in `git-svn.perl` due to duplicate option specifications that will become fatal in future Getopt::Long releases. The straightforward fix needs attention before the warning escalates.