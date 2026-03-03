# Git Mailing List Digest - 2025/05/17

## The day in brief

A moderately busy Saturday with 28 emails across 8 threads, featuring ongoing discussions about MPTCP networking support, index file optimizations, and documentation updates. The most notable developments include Junio Hamano's "What's cooking" report highlighting several graduated features, final polish on the MyFirstContribution tutorial modernization, and a contentious debate about the merits of MPTCP support in Git.

## Notable threads

### MPTCP support debate continues

Muhammad Nuzaihan's proposal to add Multi-Path TCP (MPTCP) support to Git generated significant discussion today. The v3 patch series introduced Linux-specific MPTCP support with careful fallback mechanisms, addressing earlier concerns about compatibility. However, Junio Hamano expressed skepticism about the feature's necessity, questioning whether application-level MPTCP support aligns with Git's architectural philosophy given limited ecosystem adoption. Nuzaihan defended the proposal by pointing to Apple's long-term MPTCP usage and the protocol's stability since Linux kernel 5.6. The thread remains unresolved, with the fundamental question being whether Git should lead in MPTCP adoption or wait for broader system-level support.

### Index file structure optimization discussion

Jon Forrest's exploration of potential optimizations to Git's index file structure continued with technical input from K Jayatheerth and Junio Hamano. Jayatheerth clarified that Git's index is maintained as a sorted in-memory array, with disk writes simply serializing this structure. Junio provided historical context by pointing to Git's original 2005 implementation (commit e83c5163) which established core design principles that remain relevant today. Forrest appears to be reconsidering his initial "two-file index" proposal after learning more about the current implementation's efficiency and the project's architectural constraints.

### MyFirstContribution tutorial modernization finalizes

The multi-patch series modernizing the MyFirstContribution tutorial reached its final polishing stage. Junio Hamano provided detailed feedback on commit message structure and pedagogical presentation, particularly around teaching the `UNUSED` macro pattern and repository-aware configuration access. The changes demonstrate contemporary Git practices like using `repo_config` instead of the global `git_config` and properly handling unused function parameters. With all technical changes already merged in v2, today's discussion focused on documentation clarity and commit message quality, suggesting the series is nearing completion.

### "What's cooking" report highlights recent merges

Junio Hamano's monthly "What's cooking" report provided a comprehensive overview of Git's development pipeline. Several notable changes graduated to 'master', including BSD compatibility fixes for `open_nofollow()`, Meson build system updates, and new maintenance tasks for `git maintenance`. The report also noted the renaming of `merge-tree --dry-run` to `--quiet` for better consistency with Git's CLI conventions. Ongoing topics include sparse index integration for `apply` and `add -p`, object store API cleanup, and path-based delta compression for pack-objects.

## In brief

**Bash function recognition edge cases** -- Junio Hamano and Johannes Sixt discuss final edge cases in word-diff tokenization for shell scripts, particularly around handling numbers in filenames versus mathematical decimals.

**Merge-tree --quiet follow-up** -- Kristoffer Haugsbakk explores potential enhancements to batch processing with the new `--quiet` flag, suggesting a `--format` option could simplify conflict detection workflows.

**Bitmap corruption detection test** -- A new test infrastructure was added to verify Git's handling of corrupt bitmap indexes, specifically targeting the bitmap table lookup feature with invalid commit positions and XOR offsets.

## On the radar

**MPTCP implementation debate** -- The discussion about whether Git should support MPTCP natively remains unresolved, with maintainer skepticism about ecosystem adoption contrasting with the author's arguments for the protocol's maturity and benefits.

**Index optimization exploration** -- Jon Forrest's investigation into potential index file optimizations continues, though he appears to be reconsidering the need given Git's existing in-memory sorting and other optimizations.