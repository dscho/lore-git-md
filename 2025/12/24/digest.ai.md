# Git Mailing List Digest - 2025/12/24

**The day in brief.** A busy Christmas Eve with 64 emails across 12 threads, dominated by a major feature enhancement for `git status` branch comparison that reached its sixth iteration. Notable technical discussions included a refactoring series standardizing commit array handling, a protocol violation fix for shallow clones with ref-in-want, and regression reports about lost error messages in batched reference updates.

## Notable threads

### Configurable branch comparison in `git status` reaches v6

Harald Nordgren's series adding configurable branch comparison to `git status` output progressed through three versions today (v4-v6), culminating in a polished implementation. The feature shows how the current branch differs from a configured "goal" branch (typically main/master) alongside existing tracking info. The v6 iteration renamed the configuration key from `statusGoalBranch` to `status.compareBranch` based on feedback, while maintaining all functionality from previous versions.

The implementation builds on existing branch tracking infrastructure, adding a new comparison line when the configured goal branch differs from the tracking branch. Key aspects include symbolic ref resolution (upstream/HEAD -> origin/HEAD), optimized memory management, and comprehensive test coverage. The series has evolved through extensive discussion about triangular workflows and branch naming conventions, with Junio Hamano providing guidance to use existing branch configuration mechanisms rather than introducing new assumptions.

### Refactoring commit array handling across Git

A 14-part refactoring series from René Scharfe and others standardized Git's handling of dynamic commit pointer arrays by introducing shared `commit_stack` infrastructure. The series eliminates duplicate implementations across 10+ subsystems including revision walking, logging, midx generation, and commit-graph writing. The changes are mechanical but impactful, showing a net reduction of 63 lines (247 deletions vs 184 additions) while making memory management more robust.

The new API provides initialization, growth, and cleanup helpers that replace various ad-hoc `ALLOC_GROW` patterns. The series demonstrates the API's versatility across performance-sensitive areas like bitmap generation and shallow clone operations. This foundational change should simplify future maintenance of commit-processing code throughout the codebase.

### Protocol violation in shallow clones with ref-in-want

A bugfix series addressed a protocol violation where upload-pack sent sections in the wrong order during shallow clones with ref-in-want enabled. The issue, present since 2018, caused fetches to fail with "expected 'packfile', received 'shallow-info'". The fix simply swaps two lines in upload-pack.c to send shallow-info before wanted-refs as specified in gitprotocol-v2.adoc. A new test in t5703 verifies shallow fetch behavior with ref-in-want enabled.

### Regression in batched reference update error reporting

Elijah Newren reported a regression where specific error messages (like "trying to write non-commit object") are no longer shown during batched reference updates, despite still being generated internally. Jeff King analyzed the issue, tracing it to commit 9d2962a7c44 which introduced batched updates but didn't properly propagate detailed error strings. The discussion expanded to note the regression affects fetch and update-ref operations in addition to push. A fix is in progress to enhance the ref transaction API to carry detailed error messages.

## In brief

**PID tracking for Git lockfiles** -- A feature patch adding optional PID companion files (`foo~pid.lock`) to aid debugging stale locks, with configurable component-level control via `core.lockfilePid`.

**MacOS iconv conversion fixes** -- René Scharfe simplified the Homebrew integration approach in v4 of this series, removing unnecessary component flags while maintaining version-specific activation for broken macOS versions.

**Error message formatting in `git restore`** -- Brian M. Carlson improved error output clarity by wrapping unresolved treeish names in quotes when users mistakenly use `-source` instead of `--source`.

**Whitespace check enhancements** -- Junio Hamano reviewed a patch adding `tab-between-non-ws` detection, noting it needs completion for colored diff output and whitespace fixing functionality.

**New contributor introduction** -- Andrew Chitester expressed interest in contributing through Google Summer of Code, citing experience with Magit in Emacs.