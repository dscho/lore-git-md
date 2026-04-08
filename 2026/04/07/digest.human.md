# Git Mailing List Digest - 2026/04/07

**The day in brief.** A moderately busy day with 85 emails across 20 threads, featuring significant progress on several technical fronts. Key developments include finalization of Christian Couder's promisor-remote refactoring series, resolution of an ODB transaction bugfix, and continued discussion about change-id header preservation. Junio Hamano's "What's cooking" report provides a comprehensive snapshot of the project's current state.

## Notable threads

### Promisor-remote refactoring complete

Christian Couder's 10-patch series refactoring promisor-remote handling has reached its conclusion, now incorporating all feedback from Patrick Steinhardt and Junio Hamano. The final version includes significant test improvements for cross-platform URI compatibility, standardizing path handling in `t/t5710-promisor-remote-capability.sh`. The series systematically restructures remote lookup logic to enable future auto-configuration features while improving security and cross-platform compatibility.

Key technical changes include consolidating three parallel data structures into a single `accepted_remotes` string list, hardening input validation (rejecting empty names/URLs and control characters), and decoupling config parsing. The work lays groundwork for upcoming promisor-remote enhancements while maintaining backward compatibility. With all CI tests passing and no outstanding technical concerns, the series appears ready for merging.

### ODB transaction bugfix coordinated

Justin Tobler's fix for a segfault in the ODB transaction interface has progressed through review and is now being coordinated for integration across multiple branches. The issue occurred when running `git diff` outside a repository on files exceeding core.bigFileThreshold, caused by incorrect ODB transaction handling in `index_fd()`. The solution introduces a new `hash_blob_stream()` helper to avoid transactions when only hashing is needed.

Jeff King (Peff) provided thorough review feedback, particularly around test improvements, which Justin has committed to implementing. Junio Hamano is handling a two-phase merge - first applying the standalone fix to `next` and `master`, then letting Justin's larger `jt/odb-transaction-write` topic supersede it while preserving the regression test. This demonstrates effective collaboration between author, reviewer, and maintainer to address a cross-version issue.

### Change-id header preservation debate continues

The discussion about preserving change-id headers during commit operations has evolved into a broader design debate. Nico Williams proposed a policy-based approach using project-defined rules and hooks, while Junio Hamano maintains his preference for storing such metadata in commit trailers rather than headers. brian m. carlson introduced new concerns about privacy and information leakage, arguing the feature should be opt-in.

Phillip Wood's analysis of three key scenarios (cherry-picks, commit splitting, squashing) remains the technical framework needing resolution. The thread shows movement toward potential agreement on policy mechanisms but remains divided on fundamental questions about Git's role in supporting external tool workflows. No clear resolution has emerged, with the discussion now encompassing both technical and philosophical dimensions.

## In brief

**fsmonitor test adjustment** -- Paul Tarjan follows Junio's guidance to extend split-index test coverage from t9210 to t9211, addressing the index.skipHash/split-index incompatibility consistently across both test files.

**promisor repack timestamp handling** -- Lorenzo Pegorari confirms changes for v4 of his series: fixing `localtime_r()` portability and switching timestamp storage from human-readable format to Unix epoch time for robustness.

**test modernization duplicate** -- Zakariyah Ali's v5 test modernization patch appears identical to already-merged work, with Junio noting some structural improvements but also regressions in test harness usage.

**backfill progress reporting** -- Trieu Huynh agrees with all review feedback on adding `--[no-]progress` to `git backfill`, including switching to object counting and fixing increment operator style per Git conventions.

**MIDX repacking refinements** -- Taylor Blau's incremental MIDX series receives detailed reviews from Jeff King, with consensus on technical approach but notes about future deadlock risks in pack-writing logic.

**stash argument parsing** -- Phillip Wood notes `--include-untracked` and `-m` aren't unique to "push", suggesting simpler "assume push then error" logic for Deveshi Dwivedi's series.

**fast-export ordering fix** -- Raymond E. Pasco confirms the solution for path prefix handling works with existing rename tracking, now focusing on round-trip tests relevant for future hash migration workflows.

**checkout UX clarification** -- Jesko Schwarzer proposes clarifying that "up to date" messages compare against local remote-tracking refs, with Junio welcoming patches and Ben Knoble recalling relevant fetch-time recording discussions.

**archive path handling** -- Pushkar Singh's RFC about `--prefix` path normalization sparks discussion about legitimate uses of absolute paths, with consensus forming around documenting current behavior rather than changing it.

## On the radar

**Rust by default** -- Junio's "What's cooking" notes the bc/rust-by-default-in-2.54 topic remains under discussion, with platform support concerns still unresolved.

**parallel hooks** -- The ar/parallel-hooks proposal continues developing, potentially enabling significant performance improvements for hook-heavy workflows.

**NonStop test failures** -- Randall Becker reports unexpected failures in bitmap-related tests with Git 2.54.0-rc1, suggesting possible regressions in `--git-dir` handling that may need investigation before final release.