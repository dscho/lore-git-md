# Git Mailing List Weekly Digest - 2026/04/06 -- 2026/04/12

## The period in brief

A busy week with 492 emails across 114 threads, featuring several major feature finalizations and ongoing platform compatibility discussions. Key developments include the completion of Linux fsmonitor support after 14 iterations, parallel hook execution approval, and resolution of NonStop platform `writev()` limitations. The week also saw significant progress on ODB abstraction and promisor file handling, along with debates about Git 3.0 version numbering and Rust integration timelines.

## Key developments

### fsmonitor Linux implementation finalized

Paul Tarjan's 14-iteration series implementing Linux filesystem monitoring via inotify received final approval from Junio Hamano. This brings Linux to parity with existing Windows and macOS implementations, completing cross-platform fsmonitor support. The comprehensive solution handles edge cases like watch limits, unmounts, and queue overflows while maintaining process isolation. Key technical components include non-blocking event polling, recursive watch registration, and mount point detection via `/proc/mounts`. The series demonstrates thorough engineering with attention to memory leaks, Windows pthread emulation, and test coverage, particularly around split-index compatibility issues.

### Parallel hooks ready for merge

Adrian Ratiu's parallel hook execution series (v7) received final approval after incorporating feedback from Patrick Steinhardt and Jeff King. This major feature enables concurrent hook execution with extensive configuration options:
- Global (`hook.jobs`), per-hook (`hook.<name>.parallel`), and CLI (`-j/--jobs`) controls
- Special handling for pre-push hooks with automatic output merging
- Seven commit/checkout-related hooks remain permanently serial for safety
- Comprehensive test coverage including SIGPIPE handling validation

The implementation builds on Ratiu's earlier configurable hooks work and represents a significant performance improvement for hook-heavy workflows while maintaining backward compatibility.

### NonStop platform `writev()` limitations

A platform compatibility issue emerged when Git's `writev()` optimization failed on NonStop systems due to unusually small `MAX_IO_SIZE` limits (52KB for 32-bit). After extensive discussion, Junio Hamano reverted the entire `writev()` emulation layer as a temporary solution for Git 2.54, with plans to implement proper `MAX_IO_SIZE` support post-release. Jeff King proposed a more targeted revert that would preserve scaffolding for future reactivation, but Junio opted for a clean slate approach to ensure immediate stability. The thread highlights Git's pragmatic approach when platform constraints conflict with optimization attempts.

### ODB abstraction progress

Patrick Steinhardt's 17-patch series introducing an in-memory object database source has progressed from technical implementation to final naming discussions. Junio Hamano questioned whether "in-memory" aligns with Git's existing "in-core" terminology, while Patrick argued "in-memory" will be more intuitive for future user-facing configuration. The technical implementation has already been reviewed and confirmed by Justin Tobler, representing a significant milestone in the long-running ODB abstraction effort. Meanwhile, Justin Tobler's fix for a segfault in the ODB transaction interface was coordinated for integration across multiple branches, demonstrating effective collaboration between author, reviewer, and maintainer.

### `git checkout -m` autostash finalized

Harald Nordgren's series enhancing `git checkout -m` with autostash behavior reached its final approved version (v9) after addressing all remaining review feedback. The implementation now includes support for empty conflict labels, robust stash list verification, and comprehensive test coverage (19 cases in t7201-co.sh). The last refinement renamed the internal autostash ref to `CHECKOUT_AUTOSTASH_HEAD` to comply with Git's documented rules for root-level refs. This represents a significant improvement for branch switching workflows, particularly when working with uncommitted changes.

## In brief

**Promisor-remote refactoring** -- Christian Couder's 10-patch series refactoring promisor-remote handling was completed, including test improvements for cross-platform URI compatibility and security hardening of input validation.

**Reftable compaction fix** -- Patrick Steinhardt corrected a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**Cache-tree validation fix** -- David Lin addressed an inverted condition in `cache_tree_fully_valid()` that caused incorrect validation when tree objects were missing.

**Promisor file repack handling** -- Lorenzo Pegorari's v5 series preserves debugging info in `.promisor` files during repacks, now with memory leak fixes and proper timestamp handling.

**Windows version requirements** -- Johannes Schindelin and Matthias Aßhauer's series raising Git for Windows' minimum version to 8.1 was accepted, removing legacy code paths for older Windows versions.

**French translation update** -- Jean-Noël Avila brought the French `.po` file up to date with the latest source strings.

**Rust enabled by default** -- brian m. carlson submitted a 4-patch series to make Rust support the default in Git's build system, following through on plans originally slated for 2.53.

**Git v2.54.0-rc1 released** -- Junio announced the first release candidate with 691 non-merge commits, featuring experimental `git history`, geometric repacking by default, and Rust introduction.

## Looking ahead

**Rust integration timeline** -- Build system changes to enable Rust by default are now targeted for Git 2.55, pending version numbering consensus. The technical implementation remains uncontroversial but the release strategy needs clarification.

**NonStop platform support** -- `writev()` optimizations remain deferred post-2.54 while architectural solutions are developed to handle platform's unique constraints.

**Git 3.0 planning** -- Version numbering strategy needs clarification separate from Rust integration discussion, with Junio Hamano expecting to reach 2.95 first for clearer signaling.

**Negotiation for critical branches** -- Derrick Stolee continues exploring interface design alternatives for combining must-have negotiation with existing `--negotiation-tip` after Junio Hamano suggested unification.