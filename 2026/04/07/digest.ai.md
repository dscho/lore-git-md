Here's the daily digest for April 7, 2026:

**The day in brief.** A moderately active day with 85 emails across 20 threads, featuring significant progress on several technical fronts. Key developments include finalization of Christian Couder's promisor-remote refactoring series, resolution of an ODB transaction bugfix, and continued discussion about change-id header preservation. Junio Hamano's "What's cooking" report provides a comprehensive snapshot of the project's current state.

**Notable threads**

**Promisor-remote refactoring complete**  
Christian Couder's 10-patch series refactoring promisor-remote handling has reached its final form after addressing all feedback from Patrick Steinhardt and Junio Hamano. The changes consolidate remote lookup logic to enable future auto-configuration features while improving security and cross-platform compatibility. Patch 10/10 significantly updates test URI handling, introducing `$TRASH_DIRECTORY_URL` for proper RFC 8089 compliance. The series has passed all CI tests and appears ready for merging as foundational work for upcoming promisor-remote enhancements.

**ODB transaction bugfix coordinated**  
Justin Tobler's fix for a segfault when hashing large files outside a repository is being carefully coordinated for integration. The fix splits logic in `index_fd()` to avoid ODB transactions when only hashing is needed. After thorough review from Jeff King (including test improvements), Junio Hamano is preparing a two-phase merge - first applying the standalone fix to `next` and `master`, then letting Justin's larger `jt/odb-transaction-write` topic supersede it while preserving the regression test. The collaboration demonstrates Git's meticulous approach to cross-version fixes.

**Change-id header discussion evolves**  
The debate about preserving change-id headers during commit operations has progressed toward potential agreement on a policy-based approach. Nico Williams proposed project-defined policies and hooks to handle change-id transformations, which Junio Hamano acknowledged as sensible while maintaining his preference for trailer-based metadata. The discussion remains in design phase but shows movement toward reconciling tooling needs with Git's integration boundaries. brian m. carlson later raised privacy concerns about persisting change-ids, complicating the emerging consensus.

**In brief**  

**fsmonitor test adjustment** -- Paul Tarjan follows Junio's guidance to extend split-index test coverage to t9211, ensuring consistent handling of the index.skipHash/split-index incompatibility.

**promisor repack timestamp format** -- Lorenzo Pegorari agrees to change timestamp storage in promisor files from human-readable format to Unix epoch time for robustness, while fixing `localtime_r()` portability issues.

**test modernization duplicate** -- Zakariyah Ali's v5 test modernization patch appears identical to already-merged work, with Junio noting some structural improvements but also regressions in test harness usage.

**backfill progress reporting** -- Trieu Huynh confirms plans to switch from batch to object counting in `git backfill` progress metrics and fix increment operator style per Git conventions.

**MIDX repacking reviews** -- Jeff King provides detailed feedback on Taylor Blau's incremental MIDX series, approving architectural direction while noting documentation challenges for increasingly specialized options.

**fast-export path ordering** -- Raymond E. Pasco and Jeff King confirm exporter-side ordering is correct for path deletions that are prefixes of added paths, with Pasco now writing comprehensive tests.

**On the radar**  

**Rust-by-default transition** -- Junio's "What's cooking" report notes the bc/rust-by-default-in-2.54 topic remains under discussion, with platform support questions still unresolved.

**parallel hooks proposal** -- The ar/parallel-hooks topic continues development, potentially enabling significant performance improvements for hook execution.