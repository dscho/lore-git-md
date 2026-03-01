# Git Mailing List Digest — 2025/02/10

## The day in brief

A moderately busy day with 38 emails across 19 threads, featuring significant progress on several fronts. The `the_repository` removal effort saw multiple patches advance, Junio shared a comprehensive "What's cooking" status update, and several new features were proposed for `git clean` and `git rebase`. The most notable technical discussion centered around cross-compilation build system issues and MIDX progress reporting assumptions.

## Notable threads

### Cross-compilation build system fixes

The Meson build system's handling of cross-compilation scenarios sparked an extended discussion about shell path detection. Patrick Steinhardt proposed fixes for incorrect host/target path separation, but Peter Seiderer identified potential issues with the approach based on Meson's documentation. Junio Hamano raised the broader question of whether similar fixes should be made to the make-based build system, noting that make has historically assumed matching host/target paths. The thread revealed tensions between build system parity and practical usage patterns, with indications that the Meson-specific fix may proceed while leaving make's behavior unchanged.

### MIDX progress reporting assumptions

Ayush Chandekar's RFC series adding progress reporting for MIDX verification's QSORT operation received critical feedback from Junio Hamano. The implementation tracks progress by monitoring maximum positions seen during comparisons, but Hamano questioned whether this reliably reflects qsort()'s actual progress given the algorithm's unpredictable comparison order. This fundamental design concern suggests the patch will need significant rework to either find a more reliable metric or accept and document potential inaccuracies in the progress reporting.

### `git clean` exclusion enhancements

A three-patch series from D. Ben Knoble introduced new configuration and command-line options for `git clean` to better handle excluded files. The changes add a `clean.exclude` config option (similar to the existing `-e` flag) and a `--remove-excluded` override flag. The implementation includes preparatory refactoring of pattern matching logic and careful handling of the exclusion hierarchy. The series addresses a clear use case for protecting developer-specific files while maintaining aggressive cleaning capabilities, with thorough documentation of the new features.

### Rebase's `--update-refs` gains interactive mode

A feature enhancement for `git rebase` introduced an `interactive` mode for the `--update-refs` option, allowing it to be automatically enabled for interactive rebases while remaining off for non-interactive ones. The change addresses pain points where config-set `rebase.updateRefs=true` caused unexpected ref updates in automated contexts. The patch includes comprehensive documentation updates and handles option precedence between config and command-line settings. Review feedback focused on minor documentation improvements and the need for test coverage of the new mode.

### NFSv4 packfile permissions debate

Bryan Maloney reported an NFSv4 compatibility issue where Git's temporary packfile handling causes access errors during HA filesystem failovers. The problem stems from Git creating files with O_RDWR flags but 0444 permissions, which NFSv4 re-evaluates during state recovery. Brian Carlson responded that this behavior is POSIX-compliant and suggested the solution lies in NFSv4 server improvements rather than Git changes. The discussion highlights the challenges of distributed filesystem semantics in version control operations.

## In brief

The `the_repository` removal effort saw progress with Usman Akinyemi's patch converting `builtin/update-server-info.c`, now awaiting final commit message polish per Junio's feedback. Jialuo She's path subsystem conversion received final approval, marking a major milestone in this multi-year effort.

Patrick Steinhardt's reftable decoupling series gained Windows build system improvements, with Johannes Sixt noting the need with Git-for-Windows maintainer Johannes Schindelin. Tomáš Trnka defended his promisor pack handling fix against theoretical concerns with practical performance data from production use.

Bingwu Zhang submitted a subtree bugfix for invalid HEAD handling, though Junio suggested refinements to properly distinguish between invalid and empty repository states. brian m. carlson completed portability fixes for the Thunderbird patch script with a v2 addressing final newline concerns.

## On the radar

Junio's "What's cooking" email provides a comprehensive snapshot of the project's current state, with several topics marked as needing rerolls or further review. The hash algorithm refactoring series appears ready for merging after positive feedback from Taylor Blau. The ongoing discussion about Git's pull configuration precedence continues to explore middle ground between safety and flexibility.