# Git Mailing List Digest - June 25, 2025

**The day in brief.** A moderately busy day with 68 emails across 21 threads saw several long-running efforts reach completion while new performance issues surfaced. The finalized promisor-remote protocol enhancements and stash import/export functionality stood out as major milestones, while reports of performance regressions in `for-each-ref` and `log --follow` with large repositories prompted investigation.

## Notable threads

**Promisor-remote protocol finalized**  
Christian Couder's 5-part series enhancing the promisor-remote protocol with configurable field validation reached its v5 iteration and was approved for merging. The changes allow servers to advertise additional remote attributes (like partialCloneFilter and token) via `promisor.sendFields` while enabling client-side validation through `promisor.checkFields`. The implementation builds on significant internal refactoring that replaced strvec usage with explicit struct promisor_info. Junio Hamano raised final documentation questions about field ordering and mandatory-to-understand markers, but the technical approach was solid with thorough review from Patrick Steinhardt, Karthik Nayak, and Justin Tobler. This work is part of Git's Large Object Promisors (LOP) effort and provides a foundation for future protocol extensions.

**Stash import/export merges to 'next'**  
Phillip Wood confirmed the v8 patches implementing stash transfer between repositories are ready for merging after addressing all feedback. The series introduces `git stash export` and `import` commands that preserve stash message structure and commit topology via `refs/stash-export/`. Key changes in the final version included clearer documentation, fixed memory ownership in `write_commit_with_parents()`, and additional argument validation. The implementation includes 369 lines of test coverage and reusable infrastructure like the GET_OID_GENTLY flag. Junio Hamano merged the topic to 'next', marking the successful conclusion of this long-running effort to make stashes portable across repositories.

**Performance regression in for-each-ref**  
Andrzej Dębski reported a significant slowdown in `git for-each-ref --merged` operations when processing repositories with ~65K tags, tracing the regression to commit cbfe360b140 which introduced reachability checking via `tips_reachable_from_bases`. The performance degradation scales linearly with tag count and impacts Dropbox's monorepo workflows. Kristoffer Haugsbakk forwarded the report with reproduction details showing the issue persists through Git 2.46.0. The thread awaits input from the original author of the problematic commit, as this appears to be a genuine performance regression affecting large-scale repository operations.

**Signal handling approaches debated**  
The Git daemon signal handling thread saw competing proposals emerge: Carlo Marcelo Arenas Belón's SA_RESTART toggling approach versus Phillip Wood's self-pipe alternative. Junio Hamano acknowledged both solutions could address the zombie process cleanup and race condition issues, with the self-pipe technique potentially providing more robust cross-platform behavior. The discussion revealed architectural tradeoffs - SA_RESTART requires platform-specific handling while self-pipe would restructure the event loop but eliminate those variations. The thread represents an important design decision point for reliable signal handling in the daemon.

**In brief**  
**gitk external diff bugfix** -- Johannes Sixt provided detailed technical feedback on Tobias Boesch's v6 patch fixing gitk's handling of renamed files, suggesting parsing rename info directly from diff text rather than invoking external Git commands.

**the_repository removal progress** -- Ayush Chandekar pinged an unreviewed 2-patch series migrating the prune builtin away from global state, following established patterns for this long-running effort.

**Bash completion security issue** -- Consensus emerged that the command injection vulnerability via branch names with `>` characters should be fixed in `git-completion.bash` rather than modifying refname validation rules.

**Bloom filter optimization** -- Lidong Yan's series enabling bloom filters with multiple pathspecs received review feedback from Junio Hamano requesting additional test coverage for correctness verification.

**On the radar**  
**SHA-256 by default** -- brian m. carlson indicated a v2 of this foundational series is forthcoming, delayed slightly by concurrent work on SHA-256 interop in `index-pack`.

**Case-insensitive ref pruning** -- A bug report and proposed fix emerged for `git pull --prune` failures on case-insensitive filesystems when deleting similarly-named references.