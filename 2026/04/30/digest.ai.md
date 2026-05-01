# Git Mailing List Digest - 2026/04/30

**The day in brief.** A busy Thursday with 46 emails across 13 threads saw significant progress on several fronts. The highlight was the completion of Taylor Blau's incremental MIDX repacking series, while notable discussions continued around HTTP authentication fixes, revision walking optimizations, and Windows-specific large object handling. Two bug reports surfaced regressions in `fetch --deepen` and Windows rebase behavior, with one confirmed as intended functionality.

## Notable threads

### Incremental MIDX repacking reaches completion

Taylor Blau's 16-patch series implementing incremental MIDX/bitmap-based repacking for large repositories has reached its final form. The series introduces a new repacking strategy that maintains an incrementally growing and shrinking MIDX chain to avoid periodic all-into-one repacks. The final version provides two operational modes: geometric mode creates a layered structure with older layers containing fewer, larger packs, while append-only mode preserves all prior layers unchanged. The implementation has received thorough review from both object storage (Elijah Newren) and process management (Jeff King) experts, with all technical concerns addressed. This represents a major milestone in Git's scalability for large repositories, building on years of repack machinery refactoring.

### HTTP authentication documentation finalized

Matthew John Cheetham completed the HTTP authentication bugfix series with a documentation patch clarifying the `http.emptyAuth` configuration option's behavior. The patch explicitly documents the three valid values (`auto`, `true`, `false`) with clear descriptions of each mode's authentication behavior, matching the technical implementation from earlier patches in the series. With this addition, the solution to the Kerberos (SPNEGO) authentication issue is now fully documented and ready for merging. The series has already been incorporated into Git for Windows 2.54.0-rc2.

### Revision walk interface redesigned

Mirko Faina significantly redesigned the interface for the revision walk feature in response to maintainer feedback, pivoting from `--reverse=before` to `--max-count-oldest` as the primary interface. While the core functionality remains unchanged (showing oldest commits with memory-efficient sliding window), the command-line semantics now frame the feature as a limiting operation rather than a reversal timing control. The implementation introduces a new `max_count_type` flag in `struct rev_info` to manage behavior while preserving Jeff King's underlying state machine design and Tian Yuchen's performance analysis. This represents a major interface redesign addressing Junio C Hamano's concerns about conceptual alignment with user expectations.

### Windows large object handling progresses

Johannes Schindelin's series addressing Windows platform limitations with large objects (>4GB) during cloning saw review activity, with Torsten Bögershausen noting this work supersedes an existing GitHub PR for Git for Windows. The series systematically replaces 32-bit types with 64-bit `size_t` while maintaining compatibility, with six patches submitted so far covering index-pack/unpack-objects, zlib wrapper, ODB/packfile streaming, delta header handling, test infrastructure, and regression tests. The work builds on contributions from an external developer and represents important progress for Windows users dealing with large repositories.

## In brief

**xdiff refactoring complete** -- Ezekiel Newren's series refactoring xdiff's record cleanup logic has completed its review cycle, with Phillip Wood confirming all feedback has been addressed in v6. The changes improve type safety and clarity while maintaining all optimizations from previous versions.

**git-gui repository handling** -- Shroom Moo's bugfix for git-gui's handling of bare repositories and missing worktrees reached v3, now explicitly checking parent directories when initial `--show-toplevel` fails to properly handle separated gitdir configurations.

**CI dependency updates** -- Johannes Schindelin expanded his GitHub Actions dependency update series to v6, now covering all workflow files with version bumps for msbuild setup, artifact handling, GitHub scripting, checkout, Windows SDK setup, and l10n comment posting.

**format-rev compiler warning** -- Kristoffer Haugsbakk addressed a CI-caught compiler warning in the experimental `git format-rev` command, fixing potential uninitialized variable use in the error path.

## On the radar

**fetch --deepen regression** -- Discussion continues around a regression in `git fetch --deepen` behavior introduced in Git 2.54.0, where the command incorrectly reverts fully-fetched shallow clones back to shallow state. Consensus is forming that the 2.53.0 behavior (no-op in non-shallow repos) is correct regardless of documentation wording.

**Windows rebase behavior** -- A reported change in Windows `git rebase -i` behavior when rewording multiple commits was confirmed by Phillip Wood as intended functionality, with sequential processing allowing inspection of changes for each commit.