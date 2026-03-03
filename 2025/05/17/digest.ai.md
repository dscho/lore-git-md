# Git Mailing List Digest - 2025/05/17

**The day in brief.** A moderately active Saturday with 28 emails across 8 threads, dominated by ongoing discussions about MPTCP support and documentation updates. Key developments include Junio Hamano's "What's cooking" report, final polish on the MyFirstContribution tutorial modernization, and continued debate about Git's potential MPTCP integration.

## Notable threads

**MPTCP support debate intensifies**  
Muhammad Nuzaihan's proposal to add Multi-Path TCP support to Git generated significant discussion rounds today. The technical implementation (v3) now includes Linux-specific guards and fallback mechanisms, but Junio Hamano expressed skepticism about the feature's necessity, questioning whether application-level MPTCP support aligns with Git's architecture. Nuzaihan countered with historical context about MPTCP's stability and Apple's long-term usage, though the discussion remains unresolved regarding whether Git should adopt this ahead of broader ecosystem support. The thread reveals tensions between forward-looking protocol adoption and Git's conservative compatibility philosophy.

**MyFirstContribution tutorial finalized**  
The documentation series modernizing Git's new contributor tutorial reached its conclusion today with final polish from Junio Hamano. The changes remove obsolete references to the git-mentoring list and demonstrate modern practices like repository-aware configuration access and UNUSED macro usage. Junio provided detailed feedback on commit message structure and pedagogical presentation, ensuring the tutorial accurately reflects current development practices while remaining accessible to newcomers. With all technical changes already merged, today's discussion focused on perfecting the wording before final integration.

**Index optimization exploration**  
Jon Forrest's proposal for a "two-file index" optimization prompted educational exchanges about Git's current implementation. K Jayatheerth explained the in-memory sorted array approach, while Junio Hamano suggested studying Git's original 2005 implementation (commit e83c5163) to understand the design continuity. The discussion revealed that while the proposal has theoretical merit for large repositories, existing optimizations like split index and untracked cache may already address performance needs for typical use cases. Jon appears to be reconsidering the idea's practical value after learning more about the current implementation.

**What's cooking in git.git**  
Junio Hamano's monthly status report highlighted several graduated changes including BSD compatibility fixes, Meson build improvements, and new maintenance tasks. Notable new topics include batched reference updates for fetch/push performance and continued `the_repository` removal in the replay command. The report also noted the rename of `merge-tree --dry-run` to `--quiet` for consistency with Git's CLI patterns, reflecting the project's attention to interface design even in new features.

## In brief

**Bash function recognition** -- Final review discussion about edge cases in shell script word-diff tokenization, focusing on decimal number versus filename pattern conflicts.

**Merge-tree batch processing** -- Kristoffer Haugsbakk explores potential enhancements to `--quiet` flag functionality for scripting use cases, suggesting a `--format` option.

**Bitmap corruption tests** -- New test infrastructure added to verify Git's handling of corrupt bitmap indexes, specifically targeting the bitmap table lookup feature.

## On the radar

**MPTCP implementation** -- Despite maintainer skepticism, Muhammad Nuzaihan continues refining the Linux-specific MPTCP support with improved fallback logic in v3. The architectural debate about Git's role in protocol adoption remains unresolved.