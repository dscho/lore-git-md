# Git Mailing List Digest - 2026/01/18

**The day in brief.** A moderately busy Sunday with 25 emails across 13 threads, featuring continued discussion on several major topics. The most notable developments include resolution of safety concerns around the `git-history` command's default behavior and progress on configurable branch comparisons in `git status`. Performance optimizations and test improvements rounded out the day's activity.

## Notable threads

### `git-history` safety concerns resolved

Junio Hamano has come around to Elijah Newren's position regarding the default behavior of the `git-history reword` subcommand, which updates all local branches containing the rewritten commit by default. The maintainer now agrees this approach provides better recovery options via reflog, addressing earlier concerns from SZEDER Gábor about unconventional porcelain behavior. This marks a significant step toward finalizing the series, though documentation about these tradeoffs may still need attention.

### Hook subsystem cleanup follow-up

SZEDER Gábor identified a minor cleanup opportunity in the already-approved hook subsystem refactoring series - unnecessary includes in `refs.c` that Adrian Ratiu will address in a v7 iteration. This demonstrates the thoroughness of Git's review process, where even post-approval, contributors continue to identify small optimizations. The core architectural work remains approved and ready for integration.

### Configurable branch comparisons in `git status`

Harald Nordgren has followed Junio Hamano's suggestion to make push-tracking comparisons optional via a new `status.compareBranches` configuration. The implementation allows specifying multiple branches (including `@{upstream}` and `@{push}`) for comparison while maintaining backward compatibility. A memory leak in the current implementation remains to be resolved, but the conceptual direction aligns with the thread's progression toward flexible configuration for complex workflows.

### xdiff refactoring technical discussions

The xdiff refactoring series saw several technical discussions today. René Scharfe clarified that the `ivec` growth strategy actually uses exponential growth (matching Rust's Vec), resolving one concern. He also raised macro design considerations for the C/Rust interop layer, pointing out potential pitfalls with multiple parameter evaluation. In another sub-thread, Scharfe questioned whether IVec was over-engineered for a fixed-size array case, prompting Ezekiel Newren to explain its pedagogical value for Rust migration.

## In brief

**Sparse-checkout performance tests** -- Derrick Stolee suggested concrete test improvements to demonstrate Amisha Chhajed's optimization, including duplicate pattern handling verification and a performance benchmark.

**Cygwin test fixes** -- Ramsay Jones confirmed his two test fixes now allow the full suite to pass on Cygwin, despite the platform's resource-intensive test execution.

**`git add` path handling** -- Jon Forrest acknowledged Junio Hamano's perspective on current behavior with invalid paths, reframing his concern as a minor issue now that he understands the workflow rationale.

**Test reliability improvement** -- Yushin Tian simplified t2203-add-intent.sh by using `git status --porcelain -uno` instead of grep filtering, solving an exit code masking issue more elegantly.

**New `rev-list --maximal` option** -- Derrick Stolee proposed a new flag to show only "frontier" commits, with Johannes Sixt raising interface clarity concerns about the name and documentation placement.

## On the radar

**Error reporting in batched ref updates** -- The now-merged series restoring detailed error messages may need header cleanup folded into a future hook API reapplication, as noted by SZEDER Gábor.

**Histogram diff optimization** -- Yee Cheng Chin seeks clarification on why their patch was dropped from the queue, highlighting challenges new contributors face in understanding implicit acceptance criteria.