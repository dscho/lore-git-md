# Git Mailing List Digest - 2026/03/29

## The day in brief

A moderately busy day with 47 emails across 14 threads, featuring significant progress on Taylor Blau's incremental MIDX repacking series and continued refinement of several ongoing efforts. Key developments include the completion of the `git replay --revert` feature for GitLab's Gitaly service and the introduction of dynamic mailmap toggling in `git cat-file --batch-command`. The day also saw productive discussion around test modernization and `the_repository` removal patches.

## Notable threads

### Incremental MIDX repacking reaches completion

Taylor Blau submitted the final 16-patch series implementing incremental MIDX/bitmap-based repacking, representing the culmination of multi-year work to improve Git's scalability for large repositories. The new strategy avoids periodic full repacks by maintaining a chain of MIDX layers where older layers contain fewer, larger packs while newer layers contain many smaller packs. The series introduces:

- New `--write-midx=incremental` mode with configurable compaction behavior
- Support for both geometric (compacting) and append-only repacking
- Thorough test coverage and documentation
- Careful resource management and edge case handling

The implementation builds on Taylor's previous geometric repacking work and demonstrates Git's architectural evolution to better handle monorepo-scale workloads. With all patches in the series now posted, this major feature is ready for final review.

### `git replay --revert` finalized for GitLab

Siddharth Asthana's `git replay --revert` series reached its final form with positive review feedback addressed. The feature, primarily for GitLab's Gitaly service, enables commit-by-commit reversal in bare repositories by treating reverts as merges with swapped arguments. Today's discussion confirmed three technical considerations for future iterations (extra headers handling, revision range validation, and branch creation ergonomics) while affirming the current implementation's readiness for merging.

### Dynamic mailmap toggling refined

Siddharth Asthana also advanced the mailmap toggling feature for `git cat-file --batch-command`, simplifying the interface per Junio's suggestion. The v2 patch replaces separate `use-mailmap`/`no-use-mailmap` commands with a unified `mailmap` command accepting `yes`/`no` parameters. The change maintains the caching behavior important for GitLab's long-running Gitaly processes while making the API more intuitive. Junio confirmed the design's soundness regarding mailmap data regeneration.

### Linux fsmonitor status check

Paul Tarjan provided a status update on the Linux fsmonitor implementation, confirming the v11 patches have been stable in production for months. The thread remains blocked on process questions about submission cadence and AI-assisted development norms rather than technical concerns. This highlights an ongoing disconnect between contributor and maintainer expectations around workflow practices.

## In brief

**`the_repository` removal in refs subsystem** -- Shreyansh Paliwal posted v3 of the series converting refs functions to explicit repository parameters, fixing an incorrect include and improving hash algorithm access readability while deferring timeout caching improvements.

**Test modernization review** -- Trieu Huynh's GSoC test reliability series received both administrative guidance from Junio about microproject scope and technical feedback from Tian Yuchen catching a test_grep usage bug in t4153.

**TypeScript userdiff patterns** -- Dhruv Arora's GSoC contribution adding TypeScript support to Git's diff highlighting received feedback suggesting consideration of a unified JavaScript/TypeScript pattern approach.

**SSH configuration debate continues** -- Ben Knoble reinforced the maintainer position that SSH config belongs in SSH's native files rather than Git config, citing separation of concerns in the ongoing philosophical discussion.

**`git stash create --include-untracked` RFC** -- Shabbir Bhojani proposed adding untracked file support to `git stash create` for workflows where working directory changes disrupt development tools.

**`git backfill` improvements** -- Two patches enhanced the backfill command, adding progress reporting and fixing empty repository handling, demonstrating continued refinement of this newer feature.

## On the radar

**String ownership management** -- Eric Sunshine refined his `struct str` proposal for handling string lifetimes in config.c, keeping the design minimal while addressing the specific need that sparked the discussion.

**Test modernization methodology** -- Questions arose about whether Trieu Huynh's script-generated test changes demonstrate the coding standards GSoC aims to evaluate, potentially requiring a narrower submission.