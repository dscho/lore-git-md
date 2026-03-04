Here's the daily digest for June 27, 2025:

## The day in brief

A moderately busy day with 45 emails across 15 threads, featuring significant progress on performance optimizations, several Windows-specific issues, and the debut of a proposed `git refs list` command. The bloom filter pathspec optimization series nears completion while Windows users report filesystem-related regressions.

## Notable threads

**Windows regression investigations deepen**  
Two separate Windows-specific issues saw progress today. Jörg Hohwiller reported that Git 2.46.0 works where newer versions fail with ref lock errors and clone failures, narrowing the regression window. Separately, Erwan Leroy documented "Permission denied" errors on DFS-mounted network drives in Git 2.36+, with responses pointing to a Git for Windows issue (#3825) and suggesting WSL as a workaround. These threads highlight ongoing platform-specific challenges as Git's filesystem interactions evolve.

**Bloom filter optimizations near completion**  
Lidong Yan's series optimizing bloom filters for multiple pathspecs received thorough review, with only minor test refinements remaining. The patches now correctly handle magic bit checks across all pathspec items and Junio Hamano suggested future optimizations for wildcard pathspecs. Benchmark results show the solution achieves performance parity with separate single-pathspec invocations, addressing Kai Koponen's original performance complaint.

**Git daemon signal handling refinements**  
The daemon's EINTR handling saw multiple technical discussions, with Carlo Marcelo Arenas Belón's v3 patch now addressing socket acceptance race conditions through a 3-attempt retry mechanism. While the core solution is settled, reviewers debated variable declaration styles and the interaction with SA_RESTART. Phillip Wood also proposed an alternative pipe-based child tracking implementation that could solve both the race condition and AIX recursion issues.

**New `git refs list` command proposed**  
A GSoC participant introduced an RFC for a `git refs list` subcommand, aiming to consolidate ref-related operations under a unified interface. The initial implementation exactly mirrors `for-each-ref` while establishing new infrastructure in `builtin/refs.c`. Junio Hamano questioned the long-term justification, noting users would demand feature parity between old and new commands. Jean-Noël Avila provided detailed documentation feedback to improve the man page formatting.

## In brief

**Interactive rebase worktree bug** -- Denilson Sá Maia provided detailed reproduction steps for a secondary worktree issue where `git rebase -i` incorrectly reports dirty state. Phillip Wood is investigating whether this relates to line endings or missing index refreshes.

**Promisor-remote docs polished** -- Jean-Noël Avila suggested style improvements for the finalized promisor-remote protocol documentation, recommending description-list format and consistent placeholder naming.

**Ref completion sorting patch** -- Kristoffer Haugsbakk and D. Ben Knoble reviewed style aspects of Nelson Benítez León's patch adding `completion.forRefSort` config, focusing on commit message conventions.

**Comment character selection fix** -- Ayush Chandekar's v2 patch for rebase conflict marker handling received final approval with only a typo fix needed, after addressing test improvements suggested by Phillip Wood and Christian Couder.

**Send-pack memory leak fixed** -- A 2160-byte leak in the extra_have oid array during push operations was patched, though tests don't currently exercise this code path.

## On the radar

**Clang-format line length debate** -- The discussion around setting `.editorconfig`'s `max_line_length` continues, with new data showing only 1.7% of lines exceed 120 characters. The team is weighing whether this should represent a hard limit versus a guideline.

**SHA-256 default preparation** -- Junio's "What's cooking" notes the ongoing work to prepare for SHA-256 as the default hash algorithm in Git 3.0, though brian m. carlson has indicated they won't rush the interoperability work.