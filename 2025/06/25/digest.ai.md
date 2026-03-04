# Git Mailing List Digest - June 25, 2025

**The day in brief.** A moderately active day with 68 emails across 21 threads, featuring several significant developments: the promisor-remote protocol enhancements reach final approval, a stash import/export feature is merged, and multiple performance investigations emerge. Security discussions around shell injection in Bash completion and signal handling debates in the Git daemon dominated technical discussions.

## Notable threads

**Promisor-remote protocol finalized** -- Christian Couder's 5-part series enhancing the promisor-remote protocol with configurable field validation has reached its final form after multiple review rounds. The changes allow servers to advertise additional remote attributes (like partialCloneFilter and token) while enabling client-side validation against local configuration. The implementation introduces `struct promisor_info` to replace strvec storage and includes comprehensive test coverage. Junio Hamano provided final polish feedback on protocol documentation and field ordering requirements, but the series is now approved for merging after addressing all technical concerns.

**Stash import/export merged** -- Phillip Wood's long-running stash transfer feature has been merged into 'next' after eight iterations. The implementation enables moving stashes between repositories via `refs/stash-export/` with robust validation of commit structure and message preservation. The final v8 version includes improved documentation phrasing, memory ownership fixes in `write_commit_with_parents()`, and additional argument validation. The 369-line test suite provides comprehensive coverage of the bidirectional functionality, marking the culmination of this effort to make stashes portable across repositories.

**Git daemon signal handling debate** -- Carlo Marcelo Arenas Belón's series to fix zombie process cleanup on OpenBSD and crashes on AIX has sparked a significant architectural discussion. The current approach using SA_RESTART toggling now faces competition from Phillip Wood's proposal to use the self-pipe trick instead. Junio Hamano acknowledged the self-pipe alternative could solve both the original zombie problem and a newly identified race condition between `check_dead_children()` and `poll()`. The thread has become a case study in signal handling tradeoffs, with the decision between these approaches still pending.

**Performance investigations emerge** -- Two separate performance issues surfaced today: Andrzej Dębski reported significant slowdowns in `git for-each-ref --merged` with large tag sets (65K+ tags) traced to commit cbfe360b140, while Kai Koponen identified unexpected slowness in `git log --follow` even for small files. The for-each-ref regression appears clearly linked to the `tips_reachable_from_bases` function's linear scaling with tag count, while the --follow issue may involve rename detection heuristics. Both reports include detailed reproduction cases and await maintainer attention.

**Security hardening for Bash completion** -- A vulnerability report about shell command injection via branch names containing `>` characters has evolved into a broader discussion about refname validation. While Kristoffer Haugsbakk initially suggested modifying `git-check-ref-format` to disallow problematic characters, Phillip Wood and Junio Hamano firmly established that the issue should be fixed in the completion script's failure to properly escape shell metacharacters. The consensus positions shell safety as an interface-layer concern rather than requiring changes to Git's core refname rules.

## In brief

**Bloom filter optimization** -- Lidong Yan's series enables bloom filter usage with multiple pathspecs, removing a performance limitation where combined queries (`git log -- file1 file2`) were slower than separate invocations. Junio Hamano requested additional test coverage for correctness verification.

**Case-insensitive ref pruning fix** -- ryenus reported and proposed a fix for `git pull --prune` failures on case-insensitive filesystems where similarly-named refs cause transaction rollbacks. The solution would process deletions individually rather than in batches.

**Signal handling modernization** -- Carlo Marcelo Arenas Belón's v2 series refines Git daemon signal handling with `sigaction()` and centralized SA_RESTART control, though the architectural direction may shift to Phillip Wood's self-pipe proposal.

**Clang-format improvements** -- Karthik Nayak's series removes line length enforcement from clang-format (relying on .editorconfig instead) and promotes the RemoveBracesLLVM rule from experimental to standard after a year in CI.

**SHA-256 default status** -- brian m. carlson confirmed a v2 of the SHA-256-by-default series is forthcoming, addressing Junio's feedback on this major hash algorithm transition.

**the_repository removal progress** -- Ayush Chandekar pinged an unreviewed series migrating the prune builtin away from global state, while coordinating with another GSoC participant on `is_bare_repository()` refactoring.

**GPG signature clarification** -- Junio Hamano explained that unsigned maintainer commits are expected, distinguishing between `Signed-off-by` lines and cryptographic signatures after a user reported verification failures.

## On the radar

**Batched reference updates** -- Karthik Nayak's fix-up branch addressing edge cases in bulk ref operations has been approved for merging after resolving all review feedback.

**Rebase instruction validation** -- Oswald Buddenhagen's fix for rebase state corruption with invalid formats still awaits a reroll with proper sign-off before progressing.

**Gitk external diff fix** -- Johannes Sixt provided detailed technical feedback on Tobias Boesch's v6 patch fixing renamed file handling, suggesting an alternative parsing strategy that avoids external Git commands.