# Git Mailing List Digest - 2026/05/26

**The day in brief.** A moderately busy Tuesday with 46 emails across 18 threads. Notable activity includes final refinements to the `git interpret-trailers` documentation, performance optimizations for sparse index operations in `git restore`, and a new `--refresh-stat-only` option proposal for `git update-index`. Junio's "What's cooking" report provides a comprehensive overview of the current development pipeline.

## Notable threads

### Documentation refinements for `git interpret-trailers`

Kristoffer Haugsbakk and D. Ben Knoble finalized the documentation improvements for `git interpret-trailers` key format rules. The series, now merged, provides comprehensive explanations of trailer block processing, including clear definitions of valid key formats and examples of both accepted and rejected cases. Today's exchanges focused on consolidating redundant paragraphs about default appending behavior and making references to "the trailer" more explicit. This concludes a thread that began with a bug report about strict key format parsing and evolved into thorough documentation coverage.

### Sparse index optimization for `git restore`

Derrick Stolee submitted v2 of his series optimizing `git restore --staged` to avoid unnecessary sparse index expansion when using `..` pathspecs. The key improvement extracts sparse-directory handling logic into a dedicated helper function (`try_update_sparse_directory()`) following Junio's feedback. The changes maintain the performance benefits while improving code organization, with comprehensive test coverage verifying the optimization preserves correct behavior across various scenarios. This continues Stolee's ongoing work to improve sparse checkout performance.

### New `--refresh-stat-only` for `git update-index`

A new two-patch series proposes adding `--refresh-stat-only` to `git update-index`, targeting CI/container workflows where filesystem metadata changes while content remains identical. The implementation skips content hashing while updating stat data (mtime/ctime/inode/device), with thorough test coverage and clear documentation of the tradeoffs (speed vs potential stale object IDs). The first patch fixes a preload-index bug with `--really-refresh` and assume-unchanged entries, while the second introduces the new flag. The design carefully addresses historical concerns by making this a one-shot operation rather than sticky configuration.

### `git son` command proposal

Evan Haque proposed a new `git son` command to create independent child repositories with loose parent-child relationships (distinct from submodules). The RFC includes implementation, documentation, and comprehensive tests. D. Ben Knoble provided substantive review questioning whether existing mechanisms (`git clone .`, worktrees, subtrees) already cover the use case, suggesting the proposal needs stronger justification for a new porcelain command. The discussion will likely shape whether this feature progresses beyond RFC status.

## In brief

**`git commit --fixup` message options** -- Erik Cervin-Edin's v2 series extends support for `-m`, `-F`, `-c`, and `-C` options across all `--fixup` variants, addressing prior feedback about code reuse and test hygiene.

**`push.default=simple` documentation** -- Ivan Baluta's v2 patch clarifies behavior in triangular workflows, restructuring the explanation to cover both centralized and triangular cases generically per Junio's feedback.

**Shell completion for dotfiles** -- Zakariyah Ali's v2 patch modifies path completion to hide dotfiles by default (aligning with shell conventions) while preserving explicit dot completion capability.

**Commit-reach optimization analysis** -- Kristofer Karlsson provided detailed performance analysis of alternative approaches for commit-reach.c optimizations, confirming the ENQUEUED flag solution as optimal.

**Test permission debugging** -- brian m. carlson and Junio identified root privileges in containers as the cause of unreliable `test -x` behavior, recommending tests run as unprivileged users.

## On the radar

**`diff.<driver>.process` RFC** -- Junio's latest review focuses on error handling and global state management, suggesting refinements before considering merge.

**Repository initialization refactoring** -- Patrick Steinhardt's 8-patch series centralizing object database creation appears complete with maintainer approval, marking progress in the ODB abstraction effort.