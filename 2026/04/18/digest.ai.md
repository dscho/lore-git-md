# Git Mailing List Digest - 2026/04/18 (Saturday)

**The day in brief.** A moderately busy Saturday with 29 emails across 7 threads, featuring the completion of Lorenzo Pegorari's promisor repack series, continued discussion on test suite error handling, and a new revision walk option proposal. The most notable developments were the finalization of the promisor file preservation work and Elijah Newren's partial clone optimizations being marked ready for merging.

## Notable threads

### Promisor file preservation during repacks finalized

Lorenzo Pegorari's GSoC project to preserve `.promisor` file content during repacks reached completion with the v6 series. The implementation ensures these files (which track ref-to-packfile mappings for partial clones) maintain their debugging information across repack operations, including geometric repacking scenarios. The series has undergone thorough review from Tian Yuchen and Junio Hamano, with v6 addressing all remaining concerns through:

- A new helper function `write_promisor_file_after_repack` that handles file creation and validation
- Comprehensive test coverage for both regular and geometric repacks
- Memory leak fixes and proper cleanup sequences
- Final coding style compliance with explicit header includes

The work resolves a `NEEDSWORK` comment from commit 5374a290 and demonstrates Lorenzo's growing familiarity with Git's defensive programming practices. With all substantive feedback addressed, this series appears ready for integration.

### `the_repository` removal in setup.c receives expert review

Patrick Steinhardt's series removing `the_repository` usage from `setup.c` continued to receive detailed review from Elijah Newren. While acknowledging the correctness of removing the `work_tree_config_is_bogus` global, Elijah identified that another global (`initialized`) remains, limiting multiple worktree support. The review also caught minor documentation inconsistency in `check_repository_format()` where comments still referenced `the_repository` rather than the new `repo` parameter.

These reviews demonstrate how even mechanical conversions benefit from expert scrutiny to ensure consistency across code, documentation, and long-term architectural direction. The series has already received final approval from Karthik Nayak and Junio's endorsement of splitting `setup.c` post-refactoring.

### Test suite error handling debate evolves

The ongoing discussion about enabling `set -e` error checking in the test suite saw several developments:

- brian m. carlson confirmed the mksh behavior is indeed a bug (not correct POSIX interpretation) and will report it upstream
- Jeff King (Peff) proposed a `GIT_TEST_USE_SET_E` configuration option to extend error checking beyond Bash 5+
- Junio Hamano suggested moving from version-based to capability-based detection
- Peff explored alternative approaches to catch test errors without strictly enforcing `set -e`

The discussion remains technical and solutions-oriented, now focused on practical implementation tradeoffs between strict error checking and cross-shell compatibility.

### New `--reverse=before` option proposed for revision walks

Mirko Faina proposed adding a `--reverse=before` option to revision walks, allowing reversal to happen before commit limiting (current behavior is `--reverse=after`). The change would let commands like `git log --reverse --max-count=3` return the first 3 commits rather than the last 3. Tian Yuchen's review suggested more user-centric terminology and memory efficiency improvements, leading to productive discussion about implementation strategies. The thread shows active collaboration to refine both the feature's design and its technical underpinnings.

## In brief

**Partial clone optimizations ready** -- Elijah Newren's series optimizing `git cherry` and `git grep` for partial clones is now ready for merging after removing an accidentally included design document. The changes implement batch prefetching to avoid performance penalties from individual blob fetches.

**Test suite tilde expansion fix** -- SZEDER Gábor fixed a subtle issue in `t6112-rev-list-filters-objects.sh` where unquoted tildes in grep patterns could trigger unwanted path expansion or cause segfaults in some Bash versions.

**gitk race condition investigation** -- Johannes Sixt continues investigating alternatives to Michael Warmuth-Uhl's solution for a race condition in gitk's background diff processing, seeking an approach that avoids waiting for all operations to complete.

## On the radar

**ODB abstraction work** -- Patrick Steinhardt's ongoing object database abstraction effort, which recently saw significant progress with the `setup.c` refactoring, remains a key architectural initiative to watch as it moves toward eliminating `USE_THE_REPOSITORY_VARIABLE`.