Here's the daily digest for March 1, 2026:

## The day in brief

March 1 saw steady progress across multiple fronts in Git development, with 64 emails across 24 threads. The day was marked by several long-running efforts reaching key milestones: Harald Nordgren's configurable branch comparison feature received final approval, Adrian Ratiu's config-based hooks series advanced to v3, and Tian Yuchen made significant progress on the `the_repository` removal effort. Meanwhile, new proposals and discussions emerged around path handling in `git repo-info` and editor configuration refactoring.

## Notable threads

### Configurable branch comparison approved

Harald Nordgren's multi-year effort to make `git status` branch comparisons configurable has reached its conclusion. After 28 iterations over two months (plus prior work from 2016-2018), the feature adding `status.compareBranches` configuration has received final approval from Junio Hamano. The implementation allows specifying multiple branch comparisons (`@{upstream}` and/or `@{push}`) while maintaining backward compatibility. With all technical concerns addressed and comprehensive test coverage in place, the patches are now awaiting final test review before merging to 'next'.

### Config-based hooks v3 posted

Adrian Ratiu's 12-patch series implementing config-based hooks has progressed to v3, introducing `hook.<friendly-name>.command`, `hook.<friendly-name>.event`, and `hook.<friendly-name>.enabled` configuration options. The series builds on prior work to support multiple hooks per event while maintaining filesystem hook compatibility. Notable improvements in this version include config scope tracking (global/local), disabled hook visibility in `git hook list`, and simplified callback data validation. The implementation now properly manages hook command lifecycle while preparing for future parallel execution support.

### Repository path reporting design discussion

An active design discussion emerged around path handling in the new `git repo-info` command, with multiple contributors debating the best approach for formatting paths (absolute vs relative). Lucas Seiki Oshiro's patch series introduced a `--path-format` flag mirroring `git rev-parse` behavior, but subsequent discussion explored alternatives like per-field format specifiers. The thread reached consensus on maintaining the flag-based approach for consistency, though brian m. carlson later raised Windows-specific concerns about canonicalization that may require further consideration.

### Editor configuration refactoring debate

Shreyansh Paliwal's patches to refactor editor configuration as part of the `the_repository` removal effort sparked a broader discussion about whether editor settings should remain process-wide or become repository-specific. While the initial approach kept `editor_program` as a static variable in editor.c, Tian Yuchen and Burak Kaan Karaçay argued for repository-specific handling to match Git's configuration model. Phillip Wood countered that editor preferences are inherently process-wide in practice, highlighting the tension between theoretical purity and practical usage patterns in the global state removal effort.

## In brief

**Submodule remote fix** -- Jacob Keller's patch fixing submodule fetching's assumption that all remotes are named "origin" is cleared for integration after addressing all review feedback.

**Repository statistics performance** -- Justin Tobler acknowledged performance concerns about his repository metrics feature but argued the comprehensive analysis justifies the overhead.

**Format.noprefix docs** -- Kristoffer Haugsbakk refined documentation for `format.noprefix` to better support translations while maintaining technical accuracy.

**Gitk link color** -- Wang Zichong's gitk patch adding link color customization was approved after addressing maintainer feedback about commit message style.

**Option parsing validation** -- René Scharfe's duplicate option detection patches were confirmed ready with optimized validation only during help display.

**Partial clone config** -- Alan Braithwaite proposed a `fetch.blobSizeLimit` config to enable partial clone behavior by default with clear size thresholds.

## On the radar

**GSoC project guidance** -- Phillip Wood flagged needed updates to a GSoC project description about global state removal to avoid suggesting problematic configuration handling patterns.

**Path canonicalization** -- brian m. carlson's Windows-specific concerns about path handling in `repo-info` may require follow-up design work despite earlier consensus on the flag-based approach.

The day's activity reflects Git's steady evolution across both established efforts and emerging areas, with particular energy around configuration flexibility and global state reduction. Several long-running contributions appear poised for integration while new discussions lay groundwork for future work.