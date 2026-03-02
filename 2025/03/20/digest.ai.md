# Git Mailing List Digest - March 20, 2025

**The day in brief.** A busy Thursday with 115 emails across 23 threads, dominated by major technical developments including the completion of Taylor Blau's incremental MIDX bitmap series (now ready for merging), Karthik Nayak's batched reference updates with partial failure support, and a 20-part series making Perl optional in Git's test suite. Community discussion also continued around Git Merge 2025 location planning.

## Notable threads

### Incremental MIDX bitmaps reach completion

Taylor Blau's long-running series implementing incremental MIDX bitmaps has reached its final form (v5, 14 patches) and is now ready for merging after extensive review. The feature enables efficient bitmap operations across multi-pack index layers through structured bitmaps, representing the second of three planned efforts in this area. Key improvements in this version include better documentation of object sorting semantics, renamed functions for clarity (`bitmap_num_objects_total`), memory allocation fixes, and explicit verification of MIDX layer structure in tests. Reviewers Jeff King and Elijah Newren have signed off, with all technical concerns addressed. The series maintains Git's layered bitmap design with independent *.bitmap files per MIDX layer and pack-reuse optimizations favoring older layers.

### Batched reference updates with partial failure support

Karthik Nayak introduced a significant enhancement to Git's reference handling with an 8-part series enabling batched reference updates that can partially succeed when individual updates fail. The implementation adds a `REF_TRANSACTION_ALLOW_FAILURE` flag and infrastructure to track rejected updates while still using the transaction system for optimization. All backends (files, packed, reftable) are updated to support partial failures, with comprehensive test coverage (233 lines across 8 scenarios). The changes build on Patrick Steinhardt's earlier work on batched refname checks and are particularly beneficial for reftable backend efficiency. The series has evolved through multiple iterations with naming refinements and error handling improvements based on feedback.

### Making Perl optional in the test suite

A massive 20-part series refactoring Git's test suite to remove Perl as a hard requirement has been submitted, representing a major infrastructure improvement. The changes systematically replace Perl dependencies with shell/C alternatives while maintaining test coverage, achieving 97% test pass rate (30,342/31,358 tests) without Perl. Key conversions include environment sanitization, character translation helpers, test_copy_bytes, test_readlink, and various text processing operations. A new PERL_TEST_HELPERS prerequisite clearly marks tests that still require Perl. The series touches 84 files with 471 insertions and 373 deletions, with careful attention to maintaining behavior while improving portability.

### Git Merge 2025 location discussion

Community discussion continued about the location for Git's 20th anniversary conference, with initial consensus forming around GitHub HQ in San Francisco following the traditional US/EU alternation pattern. However, concerns were raised regarding US travel restrictions and political climate making attendance difficult for some community members. Alternative locations in Canada (Toronto, Montreal, or Vancouver) were proposed as potential compromises that maintain continental rotation while avoiding US entry issues. The discussion remains ongoing but appears to be converging on San Francisco as the primary option.

## In brief

**Reftable decoupling** - Patrick Steinhardt's reftable decoupling series has resolved its final compatibility issue with Windows, confirmed by Johannes Schindelin, clearing the way for post-2.49 merging.

**Path-walk delta compression** - Derrick Stolee responded to review feedback on his path-walk delta compression series, providing additional data showing performance characteristics across different pack.window sizes and clone types.

**MinGW file handling** - A v2 patch series addresses a race condition in MinGW `O_CREAT|O_EXCL` operations that causes flaky tests on Windows, with improved error code translation for `STATUS_DELETE_PENDING` cases.

**Cruft pack handling** - Elijah Newren approved Taylor Blau's v2 series improving `git repack`'s cruft pack handling with the new `--combine-cruft-below-size` option after documentation refinements.

**Core.commentString deprecation** - Phillip Wood outlined a deprecation path for `core.commentString=auto`, proposing runtime advice messages, BreakingChanges.adoc documentation, and conditional compilation guards.

**the_repository removal** - Discussion continued in the `the_repository` removal effort, with Patrick Steinhardt suggesting improvements to NULL repository handling in `repo_config()` and better test organization.

**gitk SHA-256 support** - A revived patch series adds SHA-256 repository support to gitk, implementing basic hash parsing and display capabilities along with configuration support for automatic hash length selection.

## On the radar

**Advice system architecture** - Phillip Wood raised important questions about Git's advice system design, noting that `advise()` calls currently bypass `GIT_ADVICE` and `--no-advice` controls despite documentation suggesting they should disable all hints. This may lead to broader refactoring beyond the immediate clone operation fix being discussed.

**Atomic fetch crash** - A serious bug was reported where `git fetch --prune --atomic` crashes with a core dump when encountering lock files, traced to improper handling of closed reference transactions. Justin Tobler provided a simplified reproduction case.