# Git Mailing List Digest - 2026/05/23 (Saturday)

**The day in brief.** A moderately active Saturday with 31 emails across 13 threads, featuring final resolutions on several long-running series. Key developments include the completion of git-gui's worktree detection overhaul, approval of promisor remote configuration changes, and introduction of external notes command support. Junio's "What's cooking" report highlights recent merges and ongoing work.

## Notable threads

### git-gui worktree detection finalized

After extensive review, the 11-patch series fixing git-gui's worktree detection has reached resolution. Maintainer Johannes Sixt approved the final technical changes, including:

- Proper handling of environment variables (GIT_DIR, GIT_WORK_TREE) with core.worktree
- Corrected behavior for uncommitted changes in the blame subcommand
- Aligned argument parsing with core Git conventions
- Simplified path normalization in the browser subcommand

The series addresses edge cases dating back to 2014 while maintaining compatibility with reftable changes. With all technical questions resolved, the changes are ready for merging.

### Promisor remote configuration approved

Christian Couder's promisor remote configuration series received its final documentation polish from Kristoffer Haugsbakk, addressing minor AsciiDoc formatting issues in the security guidelines. The changes complete a significant enhancement to promisor remote functionality, including:

- URL-based auto-configuration with strict pattern matching
- Secure storage of remote fields
- Dynamic filter combination support

All technical review is complete, with the series awaiting maintainer merge alongside the previously approved LOP (Large Object Promisors) changes.

### External notes command support introduced

Siddh Raman Pant posted v3 of a series adding support for external notes display commands. The implementation includes:

- New `notes.externalCommand` configuration
- Timeout-protected subprocess communication
- Integration with log, range-diff, and describe
- 400+ lines of test coverage

The feature allows Git to fetch notes content via external programs while maintaining compatibility with traditional notes refs. Security measures include restricted configuration and input validation.

## In brief

**`--track=fetch` for checkout/switch** -- Harald Nordgren's series enabling automatic fetching when creating tracking branches reaches v13, now using shared helpers from branch.c for consistent remote resolution.

**git log --follow subtree merges** -- Miklos Vajna fixes a limitation where `--follow` wouldn't track renames across subtree merges, preserving history only when changes come from a single parent.

**Test modernization** -- A GSoC participant converts legacy path checks in t2000 to use dedicated test helpers (`test_path_is_file` etc.), removing obsolete debug code.

**Documentation typo fixes** -- Weijie Yuan offers to help restructure Andrew Kreimer's typo correction series per Junio's feedback about logical grouping and translation file handling.

**Hook documentation cleanup** -- Kristoffer Haugsbakk's mechanical improvements to hook docs receive Reviewed-by from Jean-Noël Avila, addressing formatting and cross-references.

## On the radar

**ODB refactoring** -- Patrick Steinhardt's major object database restructuring (ps/odb-source-loose) appears in Junio's "What's cooking" as an active topic, centralizing ODB creation logic.

**Linux fsmonitor** -- Paul Tarjan's in-progress Linux fsmonitor implementation (pt/fsmonitor-linux) continues development, aiming to bring Linux to parity with existing Windows/macOS backends.