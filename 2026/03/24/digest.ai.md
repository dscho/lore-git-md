Here's the daily digest for March 24, 2026:

## The day in brief

March 24 saw steady activity across Git development with 96 emails across 27 threads. The day was marked by several significant developments: the `git replay --revert` feature reached its final form ready for merging, multiple GSoC proposals entered final refinement stages, and Junio Hamano initiated a deep dive into test infrastructure robustness. Parallel hooks and `the_repository` removal efforts also saw substantial progress.

## Notable threads

**`git replay` revert capability finalized**  
The long-running series adding `--revert` support to `git replay` reached its conclusion with v5 patches from Toon Claes. This server-side history rewriting feature, particularly useful for GitLab's Gitaly service, enables reversing commits directly on bare repositories. The implementation follows the same approach as `sequencer.c`, treating reverts as merges with swapped arguments. All substantive feedback from Phillip Wood, Patrick Steinhardt, and Junio Hamano has been addressed, including improved function documentation, consolidated `revs.reverse` handling, and better error messages. The higher-level interface design discussion (flags vs subcommands) continues but is being deferred to avoid scope creep in this technically complete series.

**Parallel hooks configuration nears completion**  
Adrian Ratiu's parallel hook execution series saw extensive discussion as it approaches finalization. The patches introduce comprehensive controls for parallel hook execution through multiple configuration layers: global `hook.jobs` defaults, per-hook `hook.<name>.parallel` flags, CLI `-j/--jobs` overrides, and per-event job settings. Patrick Steinhardt provided detailed feedback on edge cases and suggested adding support for -1` to automatically use available CPU cores, which Adrian agreed to implement. The series is in its final polishing stages with all core components having maintainer approval.

**Test infrastructure robustness investigation**  
Junio Hamano initiated a deep dive into making the test suite more robust by enabling `set -e` (exit on error) mode. This was prompted by discovering that a typo in a test macro (`test_expected_success` instead of `test_expect_success`) had gone undetected. Junio methodically fixed several test scripts to work with `set -e`, addressing issues in t4032, t6002, test-lib.sh, t0008, and t7450. Jeff King later proposed an alternative approach using stderr monitoring to catch errors, leading to productive discussion about the best way to improve test reliability. The thread shows the project's commitment to understanding and fixing systemic issues.

**`the_repository` removal progresses**  
Olamide Caleb Bello submitted a substantial series moving several environment-related globals into `struct repo_config_values` as part of the ongoing effort to eliminate `the_repository`. The changes migrate variables controlling compression levels, sparse-checkout behavior, Unicode handling, and object reference warnings to per-instance storage. The mechanical but widespread changes follow established patterns from prior work in this area. The series has already undergone some review from Christian Couder and Usman Akinyemi, suggesting it's on track for integration.

**GSoC proposal season in full swing**  
Multiple Google Summer of Code proposals reached advanced stages:
- Abraham Samuel Adekunle's prioritized promisor remote fetching proposal received final polish from Christian Couder
- Francesco Paparatto submitted a thorough proposal for refactoring Git's global state
- Amisha Chhajed proposed a `git evict` command for partial clone disk management
- Jayesh Daga outlined enhancements to the new `git repo` command
- Deveshi Dwivedi's `cat-file --batch-command` extension proposal incorporated mentor suggestions

## In brief

**Commit-graph generation number fix** -- Patrick Steinhardt addressed a regression affecting dates after year 2514 by introducing `compute_generation_offset()` to properly mask commit dates to 34 bits.

**Bisect custom term handling** -- Jonas Rebmann's series to make bisect commands respect custom terms (like "old"/"new") received polish suggestions from Phillip Wood about quoting terms in output and using `test_grep`.

**`git backfill` path handling** -- Derrick Stolee's series saw edge case fixes from Patrick Steinhardt for path manipulation and unknown option handling.

**`git remote-http` segfault fix** -- K Jayatheerth's v4 patch preventing crashes when parsing refspecs outside a repository was approved by Junio.

**`diff-highlight` improvements** -- Jeff King's Perl script enhancements received final approval after addressing all review feedback.

**`git repo` help text refinement** -- Mahi Kassa's v3 patch improved subcommand-specific help output by factoring shared strings into macros.

**Root commit handling in `git replay`** -- Toon Claes' v2 patch enabling replay of root commits was approved by Junio after moving the special case into `mapped_commit()`.

**On the radar**

**String handling optimizations** -- Mateo Patino's proposal for a `strview` struct received pushback from Eric Sunshine who suggested a simpler `struct str` approach instead.

**Fast-import signature modes** -- Justin Tobler's series adding consistent invalid signature handling needs to address Junio's feedback about validation timing between import/export operations.