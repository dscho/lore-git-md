# Git Mailing List Digest — 2025/03/20

## The day in brief

A busy day with 115 emails across 23 threads saw significant progress on several fronts. The standout development was Taylor Blau's incremental MIDX bitmap series reaching final form (v5) after extensive review, while Karthik Nayak's batched reference updates with partial failure support also moved forward. Other notable activity included test suite modernization to reduce Perl dependencies, Git Merge 2025 location discussions, and multiple bugfixes.

## Notable threads

### Incremental MIDX bitmaps finalized

Taylor Blau's **incremental MIDX bitmap implementation** (v5, 14 patches) received final approvals after addressing all review feedback from Jeff King and Elijah Newren. The series introduces efficient bitmap operations across multi-pack index layers through structured bitmaps, completing the second of three planned efforts in this area. Key refinements in this version include improved documentation wording, clearer function naming (`bitmap_num_objects_total`), and explicit verification of MIDX layer structure in tests. With performance validation showing negligible impact and all technical concerns resolved, the series is now ready for merging.

### Batched reference updates with partial failures

Karthik Nayak's **batched reference updates** series (v4, 8 patches) introduces infrastructure to allow reference updates to proceed even when some individual updates fail, while still using the transaction system for optimization. The implementation includes typed transaction errors, a new `REF_TRANSACTION_ALLOW_FAILURE` flag, and comprehensive test coverage across all ref backends. Patrick Steinhardt provided final review comments suggesting minor API refinements (converting boolean parameters to flags), but the core functionality appears sound and well-tested, particularly benefiting the reftable backend's efficiency.

### Test suite modernization

A major 20-patch series progressed toward **making Perl optional** for running Git's test suite. The changes systematically replace Perl dependencies with shell/C alternatives across test infrastructure and individual tests, achieving 97% coverage without Perl. Notable conversions include environment sanitization, character translation helpers, and trace2 output scrubbing. While some tests retain PERL_TEST_HELPERS markers for complex cases, the foundation is now laid for Perl-free test execution. Eric Sunshine provided detailed review feedback ensuring test intent is preserved during conversions.

### Git Merge 2025 location debate

The **Git Merge 2025 conference location** discussion saw shifting positions, initially settling on GitHub HQ in San Francisco before community members raised concerns about US travel restrictions under current political conditions. Brian M. Carlson proposed Canadian alternatives (Toronto/Montreal/Vancouver) as a compromise maintaining North American rotation while avoiding US entry issues. The thread highlights real-world constraints affecting community participation in major events.

## In brief

**Reftable decoupling** finalized with Windows compatibility resolution as Johannes Schindelin approved the allocation fix, clearing the last blocker for Patrick Steinhardt's series to separate reftable from Git core.

**Path-walk delta compression** saw performance discussion between Derrick Stolee and Taylor Blau, with data showing variable benefits across repository types and window sizes, raising questions about the feature's value versus simpler configuration tuning.

**MinGW file handling** fixes addressed a race condition in `O_CREAT|O_EXCL` operations causing flaky tests, with v2 introducing more precise error code translation for `STATUS_DELETE_PENDING` cases.

**Atomic fetch/prune crash** was reported and reproduced, showing a core dump when encountering lock files during atomic operations, traced to a regression in reference transaction handling.

**Cruft pack handling** improvements from Taylor Blau gained Elijah Newren's approval in v2, introducing `--combine-cruft-below-size` with clarified documentation about size parameter units.

**Core.commentString deprecation** discussion advanced as Phillip Wood outlined a technical path including runtime advice messages and BreakingChanges.adoc documentation.

## On the radar

The **advice system architecture** discussion may expand beyond the immediate clone operation fix, as Phillip Wood identified a broader inconsistency where `advise()` calls bypass `GIT_ADVICE` controls despite documentation suggesting they should be suppressed.

The **Rustification effort** remains a point of contention, with Randall Becker's NonStop platform concerns still unresolved as Git Merge location discussions highlighted real-world impacts of platform support decisions.