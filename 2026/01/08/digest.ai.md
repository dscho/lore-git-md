Here's the daily digest for January 8, 2026:

## The day in brief

A moderately busy day with 50 emails across 29 threads, featuring several long-running efforts reaching completion. Key developments include final approvals for the submodule gitdir path encoding series and PID file debugging feature, plus ongoing work on xdiff refactoring for Rust compatibility. Junio's "What's cooking" report provides a comprehensive snapshot of current development activity.

## Notable threads

**Submodule gitdir path encoding finalized**  
Adrian Ratiu's long-running series to implement configurable submodule gitdir paths through the `extensions.submodulePathConfig` mechanism has received final approvals from both Patrick Steinhardt and Junio Hamano. The 8th iteration addresses all remaining feedback, including test hygiene improvements and error message formatting. Josh Steadmon confirms successful testing in production-like scenarios, clearing the way for merging this significant architectural change that enables runtime configuration of submodule paths through a four-tier fallback system.

**PID file debugging ready for merge**  
Paulo Casaretto's feature to help diagnose stale locks via `~pid.lock` files has reached its final form after addressing Junio's documentation feedback. The implementation now uses a simple boolean `core.lockfilePid` configuration following discussion with Jeff King and Patrick Steinhardt, who agreed subsystem-specific granularity was unnecessary. While PID reuse remains a theoretical limitation, the consensus is this doesn't block merging the production-ready debugging aid.

**xdiff refactoring for Rust interoperability**  
Ezekiel Newren's 10-part series refactoring xdiff internals received detailed technical review from Phillip Wood, focusing on the new `ivec` type's implementation. The review identified several areas needing attention including struct naming conventions, error handling, and type safety in vector operations. While generally approving the direction, Phillip's feedback indicates this foundational work for future Rust integration still needs refinement before merging.

**Git status shows push tracking divergence**  
Harald Nordgren checks on the status of his series enhancing `git status` to display divergence from both upstream and push tracking branches. After 17 iterations and extensive review from Junio Hamano, Ben Knoble, and Phillip Wood, the implementation appears merge-ready with all technical issues resolved. The only outstanding item is a documented quick mode edge case to be addressed separately.

## In brief

**Reftable iterator fix** -- Tsahi Elkayam corrects undefined behavior in reftable's indexed table iterator while adding a TODO about future optimization opportunities.

**Test modernization** -- Andrew Chitester improves `t1420-lost-found.sh` to use explicit path comparisons rather than fragile line counting, implementing Junio's suggested approach.

**Documentation standardization** -- Michael Lyons completes the `git-blame` man page conversion to synopsis-style format across two patches, receiving maintainer approval from Jean-Noël Avila.

**Patch-id documentation** -- Kristoffer Haugsbakk's 6-patch series improves `git-patch-id` man page clarity, standardizing terminology and documenting option interactions.

**MacOS iconv workaround** -- Torsten Bögershausen addresses character encoding conversion bugs in macOS 14/15 with a two-patch series that resets state during buffer expansion.

## On the radar

**Delta islands question** -- Martin Fick raises an apparent limitation where delta islands ignore objects not covered by bitmaps, seeking clarification on this undocumented behavior.

**Rust diff headers proposal** -- Benno Lossin suggests improving hunk headers for Rust code by including trait/impl context, sparking early technical discussion about implementation approaches.

**Git init config request** -- David Miguel Susano Pinto proposes adding `--config` support to `git init` for workflow consistency with `git clone`, though brian m. carlson suggests existing alternatives may suffice.