# Git Mailing List Digest - 2025/12/26

**The day in brief.** A moderately active day with 26 emails across 9 threads, dominated by the finalization of the multi-year hook subsystem refactoring series. Other notable activity includes progress on configurable branch comparison in `git status`, a performance optimization for `show-branch`, and discussion of a regression in git-subtree. The hook series completion stands out as today's most significant development.

## Notable threads

### Hook subsystem refactoring reaches completion

The v6 series converting Git's hook subsystem to a structured API has landed, marking the culmination of a multi-year effort. This 11-patch series standardizes all major hooks (post-rewrite, pre-push, reference-transaction, and receive-pack) on the new `hook.h` interface. Key improvements include:

- Parallel execution infrastructure with stdin streaming via `feed_pipe_fn` callbacks
- Output capture capabilities through `consume_output_fn`
- Batched processing for receive hooks (500 lines per callback)
- Sideband output preservation for server-side operations

The series has undergone six iterations with sign-offs from key reviewers including Ævar Arnfjörð Bjarmason and Emily Shaffer. The final version addresses all technical concerns and only includes minor stylistic tweaks from v5. This foundational work enables future features like config-based hooks while maintaining backward compatibility.

### Configurable branch comparison in git status nears merge

Harald Nordgren's series adding configurable branch comparison to `git status` appears ready for merging after its 8th iteration. The implementation allows setting a `status.goalBranch` configuration to track divergence from important branches beyond the immediate upstream. Today's discussion focused purely on commit organization, with Junio Hamano noting the squashed approach differs from his expectations but leaving the decision open. The technical implementation remains unchanged from v8, featuring optimized ref resolution and comprehensive test coverage across 340 lines in t6040-tracking-info.sh.

### Performance optimization for show-branch

René Scharfe submitted a performance optimization patch replacing `commit_list` with `prio_queue` in `show-branch`. The change reduces worst-case complexity from O(n²) to O(n log n) by using a min-heap for commit storage. Benchmark results show negligible impact in normal cases but a dramatic 98.6% improvement in pathological test cases. The patch touches 34 lines in `builtin/show-branch.c` and adds performance tests, maintaining existing behavior while optimizing edge cases.

## In brief

**Push group functionality volunteer** -- Usman Akinyemi volunteered to implement Junio Hamano's proposed push group functionality enhancement, which would add symmetry between fetch and push operations when using remote groups in Git config.

**Documentation feedback on LOP series** -- Jean-Noël Avila provided documentation feedback on Christian Couder's promisor remote series, suggesting back-tick formatting and questioning redundant wording in the `--filter` option description.

**Batched reference update error reporting** -- Karthik Nayak confirmed Jeff King's analysis for fixing batched reference update error reporting, agreeing to attach error details to individual failed ref updates rather than whole transactions.

**git branch --show-current -v behavior** -- A thread concluded with a decision to make `git branch --show-current -v` error out rather than silently ignore the `-flag, with Sergey Organov later exploring alternative workflows for viewing current branch status.

**pack-refs performance discussion** -- Jeff King and Brian m. carlson analyzed `pack-refs` performance bottlenecks, particularly around object verification and peeling tags, with discussion of potential reftable backend advantages.

**git-subtree regression report** -- A bug report identified a regression in `git subtree` when pushing to squashed subtree remotes, with the reporter providing a detailed reproduction case showing disconnected histories in Git 2.52.0 compared to 2.51.1.

## On the radar

**Hook subsystem future work** -- With the hook API standardization complete, watch for follow-up patches leveraging the new infrastructure for features like parallel hook execution and config-based hooks.

**Batched reference update fixes** -- Karthik Nayak's planned work to restore detailed messages for failed ref updates will need comprehensive testing across push, fetch, and update-ref operations when implemented after the New Year.