Here's the daily digest for March 1, 2026:

### The day in brief
A moderately busy day with 64 emails across 24 threads, featuring several significant developments. Key highlights include the approval of Harald Nordgren's long-running `status.compareBranches` feature, progress on config-based hooks, and continued refinement of the `git repo info` path handling design. The `the_repository` removal effort saw multiple contributions across different subsystems.

### Notable threads

**Configurable branch comparisons in git status**  
Harald Nordgren's multi-year effort to make `git status` branch comparisons configurable has reached its conclusion. After 28 iterations over two months (plus prior work in 2016-2018), the feature adding `status.compareBranches` configuration is now approved for merging into the 'next' branch. The implementation allows specifying multiple branch comparisons (@{upstream} and/or @{push}) while maintaining backward compatibility. Junio Hamano will merge it after final test review, marking a successful conclusion to this persistent contribution.

**Config-based hooks implementation**  
Adrian Ratiu's 12-patch series implementing config-based hooks (v3) was posted, allowing hooks to be specified via Git configuration (`hook.<friendly-name>.command`) rather than just filesystem scripts. The series introduces proper state management for hook callbacks, a `git hook list` subcommand with scope tracking, and comprehensive test coverage. The implementation maintains backward compatibility while providing more flexible hook configuration. Maintainer Junio Hamano has signaled readiness to proceed with the core functionality while treating parallel execution as a separate follow-up.

**Repository path reporting design**  
An extended discussion continued around path handling in `git repo info`, with multiple contributors debating the best approach for path formatting (absolute vs relative). The thread appears to be converging on using a global `--path-format` flag rather than per-field variants, accepting that users needing multiple formats must make separate calls. brian m. carlson provided valuable Windows-specific considerations regarding path canonicalization that may influence the final design.

**Global state removal efforts**  
Multiple threads advanced the ongoing `the_repository` removal project:
- Tian Yuchen posted a 3-patch series migrating encoding-related globals to repository-specific storage, incorporating Phillip Wood's feedback about configuration handling
- Shreyansh Paliwal refactored editor.c to reduce global state, sparking discussion about whether editor settings should remain process-wide
- Dorna Raj Gyawali moved the `trust_executable_bit` global into `struct repo_settings`

### In brief

**Submodule remote handling fix** -- A bugfix addressing submodule fetching's incorrect assumption that all remotes are named "origin" is cleared for integration after thorough review.

**Repository statistics performance** -- Justin Tobler acknowledged performance concerns about his repository statistics feature but argued the comprehensive analysis justifies the overhead.

**Format.noprefix documentation** -- Kristoffer Haugsbakk refined documentation about `--default-prefix` to be more translation-friendly, concluding his series.

**gitk link color customization** -- Wang Zichong's patch adding "Links" color option to gitk's preferences was approved after addressing process feedback.

**Option parser validation** -- René Scharfe's duplicate option detection was confirmed ready for merging with negligible performance impact.

**Partial clone config option** -- Alan Braithwaite proposed a `fetch.blobSizeLimit` config to enable partial clone behavior by default.

### On the radar

**GSoC project guidance** -- Phillip Wood flagged needed updates to a GSoC project description about global state removal to avoid problematic implementation patterns.

**git repo-info foundation** -- Uncertainty remains about whether the foundational `git repo info` command will be accepted, potentially stalling dependent work.