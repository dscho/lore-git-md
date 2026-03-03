# Git Mailing List Digest - 2025/06/12

**The day in brief.** A busy Thursday with 67 emails across 20 threads saw significant progress on several fronts. The stash import/export feature reached its final iteration, platform compatibility fixes dominated the discussion, and Junio introduced a new compact merge summary format. Meanwhile, architectural debates continued around batch branch operations and the `repo-info` GSoC project evolved its interface design.

## Notable threads

**Stash import/export reaches final form**  
brian m. carlson's long-running series to enable stash transfer between repositories reached its eighth and final iteration today, addressing the last two review items from Phillip Wood. The implementation now properly handles ownership semantics in `write_commit_with_parents()` and invalid argument combinations for `export`. The series introduces new `git stash export` and `import` subcommands that preserve commit chains under `refs/stash-export/`, with 268 lines of test coverage. Having incorporated feedback across seven previous versions and received positive reviews from multiple contributors (including Junio Hamano and Ramsay Jones), this appears ready for merging.

**Platform compatibility debates intensify**  
Multiple threads today tackled platform-specific build issues, with FreeBSD and Solaris taking center stage. The FreeBSD discussion established version 12 as the new baseline, removing legacy support for FreeBSD 4.x systems and debating whether to actively enforce version requirements in the build system. Meanwhile, Solaris builds were fixed by replacing sed's `-E` flag with basic regex syntax and ensuring proper newline handling in version generation scripts. These discussions revealed ongoing tensions between supporting older systems and maintaining modern code quality standards.

**Batch branch operations at an impasse**  
The architectural debate about implementing batch branch operations continued with Junio Hamano clarifying his position: while not outright rejecting Andrea Stacchiotti's porcelain approach (`git branch -f A B C X`), he insists the feature must be completely orthogonal to `--force` behavior. The thread has reached a decision point where proponents must either redesign the interface to meet these constraints or pivot to a plumbing implementation via `update-ref --stdin`. No clear path forward has emerged yet.

**`repo-info` GSoC project evolves**  
Lucas Seiki Oshiro's Google Summer of Code project to create a machine-readable repository information command saw significant interface design discussion today. Junio Hamano strongly advocated for JSON as the primary output format, questioning whether plaintext output is needed at all. When plaintext is necessary, he proposed a more robust scheme than `rev-parse`'s current behavior, using C-style quoting for problematic values. The thread also touched on eliminating `the_repository` usage in the new command, with Junio suggesting architectural changes to repository initialization code.

**Compact merge summaries proposed**  
Junio Hamano introduced a new two-patch series adding a `--compact-summary` option to merge/pull operations. The format modifies diffstat output to explicitly mark created/deleted paths during merges, addressing visibility issues in the current output. The implementation provides both command-line (`--compact-summary`) and configuration (`merge.stat=compact`) access to the new format while maintaining backward compatibility. The well-tested change appears ready for inclusion after brief review.

## In brief

**Batched reference updates polish** -- Christian Couder provided final nit-level suggestions for Karthik Nayak's patch handling directory/file conflicts in batched reference updates, focusing on terminology consistency ("D/F conflict" vs "F/D conflict").

**Rebase workflow debate** -- Nico Williams advocated for rebase-only workflows in response to an RFC about merge-heavy workflows, sharing a conflict bisection script as an alternative approach.

**Force-push safety discussion** -- Aditya Garg found D. Ben Knoble's custom `git-div` visualization and strict force-push aliases to be a satisfactory solution to his original concerns about accidental force-pushes.

**BSS initialization guidance** -- Junio Hamano added explicit documentation in CodingGuidelines advising against redundant zero/NULL initialization of globals when BSS will handle it, with positive review from Christian Couder.

**Byte-order handling fixes** -- Sebastian Andrzej Siewior's series modernizing Git's byte-order handling saw patch 1 (a big-endian regression fix) accepted for Git 2.50, while optimization patches 2-6 were deferred to the next cycle.

**Localization updates** -- Jiang Xin submitted translation updates for Git 2.50.0, adding Irish as a new language (ga.po) and updating 8 existing translations, with Traditional Chinese (zh_TW) added in a follow-up.

## On the radar

**`rebase--` RFC** -- The thread about merge-heavy workflow tooling has shifted from technical implementation to a philosophical debate, with Nico Williams' rebase advocacy introducing new perspectives that may influence future discussion.

**Mailmap bugfix** -- A fix for mailmap handling in `git cat-file` is technically ready but established an important precedent about review attribution across platforms when Junio confirmed Christian Couder's GitLab review was valid for a mailing list patch.