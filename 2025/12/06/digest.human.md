# Git Mailing List Digest - 2025/12/06

**The day in brief.** A moderately active Saturday with 49 emails across 11 threads, featuring significant progress on MIDX compaction, security hardening of tempfile handling, and ongoing discussions about submodule path encoding. Taylor Blau's 17-patch MIDX compaction series leads today's technical contributions, while René Scharfe's security series eliminating insecure `mktemp(3)` usage reaches completion.

## Notable threads

**MIDX compaction series lands** -- Taylor Blau introduces a 17-patch series implementing MIDX layer compaction, a key component of Git's incremental repacking strategy for large repositories. The series allows combining adjacent MIDX layers while preserving bitmap validity by maintaining pseudo-pack ordering. Notable aspects include relaxing lexicographic pack ordering requirements (patch 9/17), introducing overflow-safe arithmetic helpers (patch 10/17), and adding bitmap support during compaction (patch 17/17). The implementation includes thorough test coverage in t5335 and builds on Taylor's previous geometric repacking work.

**Secure tempfile handling completed** -- René Scharfe's security series eliminating insecure `mktemp(3)` usage reaches completion with v2 patches that: introduce `git_mkdtemp()` (1/5), migrate Windows compatibility code (2/5), remove deprecated functions (3/5), ban `mktemp(3)` (4/5), and eliminate the now-redundant `gitmkdtemp()` wrapper (5/5). The changes proactively remove vulnerable patterns before they can be exploited, following Git's security hardening practices. Jeff King provided review input throughout the thread, particularly on build system considerations.

**Submodule path encoding configuration** -- The submodule gitdir path encoding thread sees final alignment on configuration approach after Patrick Steinhardt and Junio Hamano resolve remaining questions. They agree the feature will use pure runtime configuration via `/etc/gitconfig`, rejecting build-time options as unnecessary complexity. Patrick proposes an atomic enablement command to ensure config consistency when activating the extension, addressing edge cases where manual enablement could leave repositories in an invalid state. The discussion confirms the architectural principle of strict config authority when the extension is active.

**Promisor pack performance optimization** -- Aaron Plattner's optimization for promisor pack handling sees review from Jeff King, who suggests splitting out an `OBJ_NONE` handling fix as a separate patch. The change dramatically reduces processing time for large promisor packs (76 minutes -> 2 minutes in one case) by skipping unnecessary blob decompression. Discussion now focuses on whether skipping hash verification for all object types (not just blobs) might impact corruption detection, with Plattner proposing three potential resolution paths in his latest email.

**Clar framework updates** -- Patrick Steinhardt's clar test framework update series reaches v2 with three patches improving integer handling in assertions and addressing test output formatting. The changes introduce type-safe integer comparisons using `intmax_t`, add relative comparison macros (`cl_assert_lt_i` etc.), and fix TAP output formatting. A new patch responds to Junio Hamano's observation about trailing blank lines in test output by adding a `.gitattributes` rule to accommodate them. Jeff King's review input helped resolve type safety questions around format string handling.

## In brief

**Reftable test assertions** -- Patrick Steinhardt demonstrates the clar framework's new type-safe assertions by converting 11 checks in reftable unit tests to use macros like `cl_assert_gt_i()`.

**Object database leak fix** -- Patrick Steinhardt corrects a 192-byte memory leak during git-grep operations involving submodules by fixing cleanup ordering in `odb_free()`.

**Windows symlink behavior** -- Junio Hamano and Johannes Schindelin align on documenting Windows symlink handling changes in terms of POSIX standards rather than Linux-specific behavior.

**MIDX documentation fixes** -- Taylor Blau corrects outdated MIDX bitmap compatibility notes and aligns command synopses with actual `-h` output as preparatory work for his compaction series.

**Structured data version control** -- The thread evaluating Git modifications for structured data versioning considers existing tools (DVC, Dolt, Fossil SCM) as alternatives, with Cedric Sodhi analyzing their architectural tradeoffs.