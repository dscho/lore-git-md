# Git Mailing List Digest - 2026/01/18

## The day in brief

A moderately busy day with 25 emails across 13 threads, featuring significant progress on the `git-history` command's design philosophy, continued refinement of the hook subsystem, and several performance optimizations nearing completion. The most notable development is Junio Hamano's endorsement of the `git-history reword` command's multi-branch update behavior, resolving a key design debate.

## Notable threads

### `git-history` command design consensus

Junio Hamano has come around to supporting the `git-history reword` command's default behavior of updating all local branches containing the rewritten commit, aligning with Elijah Newren's position. This resolves a key philosophical debate about whether Git's traditional single-branch focus should yield to multi-branch awareness in new commands. The maintainer now accepts that reflog-based recovery justifies this more expansive default behavior, though documentation about these tradeoffs may still need attention. This moves the thread closer to resolution after SZEDER Gábor's earlier concerns about unconventional porcelain behavior.

### Hook subsystem refinements

The now-approved hook subsystem refactoring series received minor polish suggestions, with SZEDER Gábor pointing out unnecessary includes in `refs.c` that can be cleaned up. Adrian Ratiu confirmed these will be addressed in a v7 iteration. This demonstrates the thoroughness of Git's review process, where even post-approval, contributors identify small optimizations. The core architectural work remains unchanged and ready for integration.

### Configurable branch comparisons in `git status`

Harald Nordgren has implemented Junio's proposed `status.compareBranches` configuration, allowing customizable branch comparison in status output. The v26 iteration builds on previously merged push-tracking functionality, now permitting multiple branch specifications. While conceptually sound, the implementation currently has a memory leak that needs resolution. The thread continues to balance immediate workflow needs with long-term configuration design, with emerging consensus that triangular workflow practitioners often prefer binary state information over detailed counts.

### xdiff refactoring technical discussions

The xdiff refactoring series saw several technical discussions progress: René Scharfe corrected his initial assessment of the growth strategy (confirming it's exponential, not linear), while also raising concerns about macro pitfalls in the `ivec` implementation. Ezekiel Newren defended using IVec even for fixed-size arrays as part of the Rust interoperability strategy, highlighting tensions between immediate C needs and Rust migration goals. These exchanges demonstrate the careful attention being paid to memory management patterns that must work well in both C and Rust contexts.

## In brief

**Hook subsystem header cleanup** -- SZEDER Gábor identified unnecessary includes in `refs.c` from the now-reverted hook API changes, suggesting they be removed in a future reapplication of the series.

**Sparse-checkout optimization validation** -- Derrick Stolee provided test case suggestions to Amisha Chhajed for demonstrating her O(n log n) pattern sorting optimization, including stress testing with 1000 duplicate patterns.

**Cygwin test fixes confirmed** -- Ramsay Jones verified his patches fixing t9700 and t0610 test failures now allow the full test suite to pass on Cygwin, despite the platform's resource-intensive test execution.

**`git add` behavior discussion concludes** -- Jon Forrest acknowledged Junio Hamano's perspective that current `git add` behavior with invalid paths represents intentional design rather than a bug, winding down the thread.

**Test reliability improvement** -- Yushin Tian simplified t2203-add-intent.sh by using `git status -uno` instead of grep filtering, solving an exit code masking issue more elegantly.

## On the radar

**`rev-list --maximal` proposal** -- Derrick Stolee's new option to show frontier commits is under discussion, with Johannes Sixt raising interface clarity concerns about the name and documentation placement.