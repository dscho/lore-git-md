Here's the daily digest for April 28, 2026:

## The day in brief

A moderately busy day with 57 emails across 14 threads, featuring several significant developments. The standout items include Johannes Schindelin's Windows compatibility fixes for large object handling, Kristoffer Haugsbakk's new `git format-rev` command, and the conclusion of the long-running autostash feature series. Junio Hamano also weighed in on multiple threads, providing final approvals and interface design feedback.

## Notable threads

### Windows large object handling fixes

Johannes Schindelin submitted a comprehensive 6-patch series addressing Windows platform limitations with objects larger than 4GB. The changes systematically replace 32-bit `unsigned long` with 64-bit `size_t` in critical paths (index-pack, unpack-objects, zlib wrapper, ODB streaming, and delta handling) to prevent truncation during cloning operations. The series includes new test infrastructure (`test-tool synthesize pack`) and regression tests to verify the fixes. This represents a significant improvement for Windows users working with large repositories, though additional work remains for other code paths like `cat-file`.

### New `format-rev` command

Kristoffer Haugsbakk pivoted from extending `git name-rev` to introducing a new experimental `git format-rev` command in v3 of his series. The command provides on-demand pretty formatting of revisions with two input modes: `--stdin-mode=revs` (one revision per line) and `--stdin-mode=text` (finding commits in freeform text). The implementation shares infrastructure with `name-rev` and supports all standard pretty formats and notes display. Marked as EXPERIMENTAL to allow interface refinement, this looks like a useful addition for workflows needing to format commits on demand rather than walking commit graphs.

### Autostash feature concludes

After 20 iterations (14 pre-merge and 6 post-merge), the autostash feature for `git checkout -m` is now complete. The final patches added customizable conflict marker labels, silent operation capability, comprehensive lockfile cleanup, and robust test coverage. Phillip Wood provided the last round of review feedback, confirming the technical soundness while noting opportunities for future polish. The implementation demonstrates Git's thorough review culture, with all edge cases addressed across multiple subsystems (`builtin/checkout.c`, `sequencer.c`, `builtin/stash.c`, `xdiff/xmerge.c`).

### `--reverse=before` interface debate

Junio Hamano strongly advocated for Johannes Sixt's `--max-count-oldest` approach over the current `--reverse=before` design in Mirko Faina's patch series. While the technical implementation is sound, Junio argues the feature should be framed as controlling when limits apply rather than when reversal occurs. This philosophical disagreement about interface design represents a key decision point for the series as it nears completion. Junio also flagged test fragility due to hardcoded SHA-1s and minor coding style issues needing cleanup.

### HTTP authentication documentation

Matthew John Cheetham proposed documentation improvements for the `http.emptyAuth` configuration option to complement the recently merged Kerberos authentication fixes. The suggested changes would explicitly list the three possible values (`auto`, `true`, `false`) with clear descriptions of each behavior. This documentation polish addresses Junio's earlier observation that the current docs are unclear about allowed values and their effects.

## In brief

**Remote group push compiler fixes** -- Junio C Hamano submitted a final polish for Usman Akinyemi's remote group push feature, fixing `-Werror=sign-compare` warnings by adjusting variable types in `builtin/push.c` and `remote.h`.

**Index-pack I/O optimization approved** -- Junio gave final approval to Scott Bauersfeld's patch increasing I/O buffer sizes in `index-pack`/`unpack-objects` from 4KB to 128KB, noting 10-11% faster clones on FUSE filesystems.

**Windows geometric repack deadlock fix** -- Johannes Schindelin addressed a Windows-specific deadlock in geometric repacking during `git maintenance`, with Derrick Stolee confirming LGTM for the fix that properly sets `odb_to_close`.

**Line-log diff pipeline refactoring** -- A 3-patch series refactored `git log -L` to integrate with Git's standard diff output pipeline, fixing issues with pickaxe and diff-filter while enabling new format support.

**Reintegrate script error reporting** -- Erik Cervin-Edin fixed incorrect warning message redirection in the Reintegrate script, with Tian Yuchen confirming the technical correctness of the change.

## On the radar

**Tarball reproducibility discussion** -- The thread exploring `git archive` improvements for Debian packaging workflows continues, with brian m. carlson proposing a versioned format approach to handle platform variations in compression and timestamps.

**Checkout --fetch proposal rejected** -- Junio Hamano firmly rejected Harald Nordgren's proposal to add `--fetch` behavior to `git checkout`/`git switch`, arguing it encourages poor workflow practices despite the technical implementation being sound.