# Git Mailing List Digest - 2025/06/12

**The day in brief.** A busy Thursday with 67 emails across 20 threads saw significant progress on several fronts. The stash import/export feature reached its final iteration, platform compatibility fixes dominated the discussion, and Junio proposed a new compact summary format for merges. Notable technical discussions included FreeBSD version support policies and Solaris build fixes.

## Notable threads

**Stash import/export reaches final form**  
brian m. carlson submitted the eighth and final version of the stash import/export series, addressing the last two review items from Phillip Wood. The implementation now properly handles ownership semantics in `write_commit_with_parents()` and invalid argument combinations for `export`. The series introduces new `git stash export` and `import` subcommands that enable bidirectional transfer of stashes between repositories, storing commit chains under `refs/stash-export/`. With 268 lines of test coverage and positive reviews from multiple contributors, this long-running series appears ready for merging.

**FreeBSD version support debate**  
A multi-email discussion established FreeBSD 6.0 as the new minimum supported version, removing several legacy build settings (NO_MEMMEM, NO_UINTMAX_T) that were only needed for FreeBSD 4.x systems. The conversation evolved into broader platform support policies, with Junio suggesting Documentation/technical/platform-support.adoc as the venue for formalizing version requirements. Carlo Arenas later proposed actively enforcing version 12+ for `memmem()` usage at the build system level, which Junio supported as a defensive approach.

**Solaris sed compatibility fixes**  
Multiple contributors diagnosed and fixed build issues on Solaris where older sed implementations don't support the -E flag for extended regex. Junio provided a portable solution using basic regex syntax in generate-configlist.sh and ensured proper newline handling in GIT-VERSION-GEN. The changes maintain functionality while working across platforms, avoiding the need to require GNU sed. Follow-up discussion explored broader shell script portability best practices around `echo` vs `printf`.

**Compact merge summaries proposed**  
Junio introduced a new `--compact-summary` option for merge/pull operations that modifies diffstat output to explicitly mark created/deleted paths. The two-patch series extends both the command-line interface and `merge.stat` configuration to support the new format, which shows new files with "(new)" suffixes. The implementation builds cleanly on existing diffstat infrastructure while maintaining backward compatibility, with comprehensive test coverage verifying both CLI and config-driven behavior.

**Byte-order handling refactoring progresses**  
Sebastian Andrzej Siewior's series modernizing Git's byte-order handling saw patch 1 (a big-endian regression fix) accepted for Git 2.50, while the remaining optimization patches were deferred to the next cycle. Discussion revealed that modern compilers already optimize shift-based byte swaps to bswap instructions, making some performance motivations less critical than initially thought. The series continues to address platform-specific detection and MSVC support.

## In brief

**Batch reference updates** -- Christian Couder provided stylistic review feedback on Karthik Nayak's patch for directory/file conflict handling in batched reference updates, focusing on terminology consistency ("D/F conflict" vs "F/D conflict").

**Force-push safety** -- Aditya Garg and D. Ben Knoble discussed workflow solutions for safe force-pushing, with Garg expressing interest in Knoble's custom `git-div` visualization script and strict force-push aliases.

**BSS initialization guidance** -- Junio's patch adding CodingGuidelines advice against redundant zero/NULL initialization of globals received a positive review from Christian Couder.

**Mailmap bugfix** -- Junio confirmed Christian Couder's Reviewed-by trailer was valid despite the review occurring on GitLab, establishing precedent for cross-platform review attribution.

**FreeBSD build cleanup** -- Brad Smith removed obsolete FreeBSD 4.x compatibility settings from config.mak.uname, establishing FreeBSD 6.0 as the new baseline.

**the_repository removal** -- Lidong Yan's patch replaced two remaining `the_repository` uses in `run_builtin()` with the passed `repo` parameter, continuing the project's broader effort to eliminate global repository access.

**Localization updates** -- Jiang Xin submitted translation updates for Git 2.50.0, including a new Irish (ga.po) translation and updates for 8 existing languages, totaling over 33,000 lines of changes.

## On the radar

**Batch branch operations debate** -- The architectural discussion about implementing batch operations in `git branch` vs `update-ref` continues, with Junio clarifying his objection to coupling the feature with `--force` behavior. Andrea Stacchiotti appears open to redesigning the interface but awaits clearer community demand.

**rebase-- RFC** -- Nico Williams advocated for rebase-only workflows in response to the `rebase--` tool proposal, shifting discussion toward workflow philosophy rather than technical implementation.

**git repo-info GSoC** -- The new repository information command saw interface design discussions, particularly around plaintext output format safety and `the_repository` usage, with Junio strongly advocating for JSON as the primary output format.