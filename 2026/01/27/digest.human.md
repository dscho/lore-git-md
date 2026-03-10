# Git Mailing List Digest - 2026/01/27

**The day in brief.** A moderately busy Tuesday with 40 emails across 15 threads, featuring final polish on several major efforts including the hook subsystem refactoring and MIDX compaction, plus UI improvements for `git add -p` and `git switch`. Junio released Git v2.53.0-rc2 and posted a "What's cooking" report tracking the project's current state.

## Notable threads

### Hook subsystem refactoring nears completion

The long-running hook subsystem refactoring by Adrian Ratiu and Emily Shaffer is in its final polishing phase after addressing all major technical concerns. Today's discussion focused on naming conventions for parallel execution helpers (`child_is_spewing_error` vs `child_is_sending_output`) and clarifying the async muxer behavior in receive-pack hooks. The series has maintainer approval and is effectively ready for merging pending these last documentation refinements, with Junio playfully suggesting names while reviewing the parallel child process infrastructure.

### MIDX compaction series refined

Taylor Blau's multi-pack-index compaction series received detailed review from Patrick Steinhardt on version compatibility handling between MIDX v1 and v2 formats. The discussion examined pack name comparison logic and error checking redundancy, with Patrick suggesting alternative approaches while acknowledging the core version compatibility approach is sound. This represents the careful final review of a performance-sensitive feature that will improve scalability for large repositories.

### `git add -p` navigation improvements

Abraham Samuel Adekunle proposed adding context-sensitive file navigation ('<' and '>') to `git add -p` when all hunks in a file are decided. Junio approved the core design but raised concerns about backward compatibility and suggested making the feature opt-in. The patch would modify `add-patch.c` to track file positions and introduce new prompt states, representing a quality-of-life improvement for interactive staging workflows. Discussion continues on whether to adjust the patch application timing and how to handle the UI changes.

### `git switch` advice message improvements

Junio C Hamano sent a two-patch series to improve `git switch`'s advice when encountering ambiguous remote branch names. The changes ensure the command suggests `git switch --track` rather than `git checkout --track` when appropriate, matching the command actually used. The refactoring introduces an enum to track command types through the checkout machinery, allowing customized advice while maintaining shared implementation. Review discussion focused on test design details, with Kristoffer Haugsbakk suggesting potential simplifications to the test cases.

## In brief

**Git v2.53.0-rc2 released** -- Junio announced the second release candidate with 451 non-merge commits since v2.52.0, featuring new features, performance improvements, and numerous bugfixes across all major subsystems.

**What's cooking report** -- Junio's status update noted new documentation patches, graduated Windows symlink support, and ongoing work like Patrick Steinhardt's ODB refactoring, while listing several stalled topics for potential removal.

**git-gui tab alignment fixes** -- A bugfix series addresses tab display inconsistencies between git-gui and gitk on Windows, with the maintainer Johannes Sixt rejecting text-modifying solutions in favor of Tk-native tab size configuration.

**`string_list_sort_u` RFC** -- Amisha Chhajed's proposal for a combined sort+deduplicate utility identified edge cases in fetch.c and help.c where order preservation matters, prompting reconsideration of the API design.

**`git last-modified` bugfixes** -- A completed set of fixes for input validation, error handling and memory leaks in this command was acknowledged by Junio, concluding a thread that also sparked discussion about "revision" terminology.

**Documentation linter updates** -- Final patches from Jean-Noël Avila and D. Ben Knoble completed work to handle conditional blocks in documentation, following patterns established for `git shortlog`.

## On the radar

**ODB refactoring** -- Taylor Blau requested a delay in merging Patrick Steinhardt's object enumeration API changes to allow more review time for mtime-related modifications in the 16-part series.

**`ls` alias proposal** -- A discussion began about adding 'ls' as shorthand for 'list' subcommands, citing CLI convention consistency, though Junio expressed skepticism without stronger Git-internal precedent.