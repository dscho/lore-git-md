Here's the daily digest for September 26, 2025:

## The day in brief

A busy day with 87 emails across 31 threads, featuring significant progress on several fronts. Key highlights include the completion of Karthik Nayak's reftable fsck validation series, final approval for Phillip Wood's `git add -p` hunk splitting changes, and an important discussion about Git's patch review workflow bottlenecks. The Git PLC also published its annual financial and governance report.

## Notable threads

### Reftable fsck validation reaches completion

Karthik Nayak's v4 series introducing consistency checks for the reftable backend received maintainer approval and is now queued for merging. The final version simplifies validation by checking only tables listed in the stack rather than scanning directories, eliminating race conditions. Key improvements include stricter parsing of 'tables.list' (now requiring trailing newlines), non-fatal table name validation (emitting warnings rather than errors), and proper error mapping between reftable and Git's fsck systems. The series represents a significant step toward feature parity between reftable and files backends.

### `git add -p` hunk splitting behavior finalized

Phillip Wood's changes to make split hunks explicitly undecided in `git add -p` received final administrative approval from Junio Hamano. The thread concluded with discussion of two remaining UI limitations that could be future work: inability to revisit decided hunks and lack of visual state indication. The core change - marking split hunks as `UNDECIDED_HUNK` in `add-patch.c` - remains unchanged from earlier versions and is now cleared for merging.

### Patch workflow bottlenecks examined

Junio Hamano initiated a detailed discussion about optimizing Git's patch review workflow, identifying the transition from 'seen' to 'next' as the most common bottleneck. Analyzing four recent merged topics, he showed cases where patches waited days in 'seen' despite being technically ready. The thread explores potential solutions including distributed maintainer responsibilities, while acknowledging the challenge of compressing early review cycles where technical feedback is most valuable.

### Annual PLC report published

The Git Project Leadership Committee shared its yearly financial and governance update, reporting $99k in reserves (up $6k from 2024). Notable details include transition to free GitHub Pages hosting for git-scm.com, standard trademark renewals, and questions about future spending priorities like self-funding Outreachy internships. The report emphasizes SFC's non-involvement in technical decisions while maintaining oversight of legal and financial matters.

## In brief

**Packfile store refactoring complete** -- Patrick Steinhardt's series centralizing packfile management in `struct packfile_store` received final approval after addressing all feedback in v6. The changes migrate packfile-related state from `object_database` and update ~50 call sites.

**xdiff modernization concludes** -- Ezekiel Newren's 12-part series modernizing xdiff internals reached completion with bool conversion of the `changed[]` array. The series eliminated redundant structures, improved type safety, and introduced clearer constants for line classification states.

**Atomic `git replay` updates** -- A patch series finalizing atomic reference updates for `git replay` transitioned from RFC to implementation-ready state. The changes make atomic behavior default while maintaining backward compatibility via `--output-commands`.

**Windows process handling regression** -- A Git Bash regression preventing GUI process detachment when launched via Makefiles was narrowed to interactions with `cmd //c` in version 2.51.0. Johannes Schindelin confirmed the simpler `git gui &` case works in preview builds.

**Documentation formatting fixes** -- Ramsay Jones submitted a 4-part series standardizing technical documentation formatting, addressing AsciiDoc warnings in files like commit-graph.adoc and sparse-checkout.adoc. Discussion continues about header style conventions.

**Hook subsystem refactoring progresses** -- Adrian Ratiu's hook modernization saw extensive review of patches 3-6, focusing on parallel execution semantics, stdin feeding efficiency, and output handling in the new `hook.h` infrastructure.

## On the radar

**Rust boolean config implementation** -- The libgit-rs boolean config parsing discussion took an architectural turn, with maintainers insisting the implementation should wrap Git's native `git_configset_get_bool()` through FFI rather than reimplementing in Rust.

**Commit graph backend handling** -- Patrick Steinhardt's ODB abstraction effort remains paused pending response to Junio's critique about whether commit graphs should be backend-specific optimizations or fundamental cross-backend metadata.

**`push.default=simple` documentation** -- Post-merge analysis revealed divergence between original design and current behavior of `push.default=simple`, with the mode now allowing same-name pushes without upstream configuration in many cases.