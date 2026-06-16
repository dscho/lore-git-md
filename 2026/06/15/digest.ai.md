Here's the daily digest for 2026/06/15:

**The day in brief.** A busy Monday with 111 emails across 28 threads, featuring significant progress on several major efforts including Patrick Steinhardt's `git history drop` series reaching v6, Johannes Schindelin's Windows large object handling patches getting queued, and Harald Nordgren's `git branch --delete-merged` completing its 15-iteration journey. Documentation and testing improvements dominated the smaller patches.

**Notable threads**

**`git history drop` reaches completion**  
Patrick Steinhardt's 10-patch series implementing the `git history drop` subcommand has reached v6 with all technical concerns addressed. The feature allows removing a commit from history while replaying its descendants, requiring significant refactoring of Git's reset machinery along the way. The final version fixes a dry-run inconsistency identified by Junio Hamano and includes 537 lines of test coverage. This represents both a major new capability for the experimental `git history` command and important infrastructure improvements to Git's reset functionality.

**Windows large object handling queued**  
Johannes Schindelin's 7-part series addressing Windows platform limitations with objects larger than 4GB has been acknowledged by Junio Hamano and queued for potential inclusion. The patches systematically convert `unsigned long` to `size_t` for object sizes across multiple subsystems, resolving conflicts with the recently merged `ps/odb-source-loose` series. With substantive reviews from Patrick Steinhardt incorporated and maintainer approval signaled, this important compatibility work appears ready for the next development cycle.

**`git branch --delete-merged` finalized**  
Harald Nordgren's long-running series adding safe branch cleanup via `git branch --delete-merged` has reached v15 with all planned features complete. The implementation now includes per-branch opt-out configuration, dry-run previews, and comprehensive safety mechanisms after 15 iterations of refinement. Phillip Wood provided detailed final reviews focusing on documentation clarity and implementation polish. The mature series demonstrates Git's iterative development process yielding a robust solution to a common workflow need.

**In brief**  

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Compiler macro modernization** -- Dominik Loidolt's series updating compiler compatibility macros in `compat/posix.h` has been acked by both Patrick Steinhardt and Junio Hamano for integration.

**Path formatting standardization** -- A GSoC participant's series introducing `append_formatted_path()` for consistent path handling has addressed all review feedback and appears ready for integration.

**Git Bash GNUPGHOME bug** -- Will Flowers reports a Windows-specific path handling issue where Git Bash incorrectly concatenates rather than replaces the GNUPGHOME environment variable.

**On the radar**  

**`git rebase --squash` proposal** -- Harald Nordgren's feature to fold commit ranges while preserving messages has prompted discussion about whether this belongs in rebase or should be a `git history` subcommand.