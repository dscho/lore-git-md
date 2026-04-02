Here's the Git mailing list digest for April 1, 2026:

### The day in Brief
April 1st brought a mix of serious technical discussions and April Fools' humor to the Git mailing list. While several significant technical threads progressed - including fsmonitor stabilization, ODB transaction interfaces, and const-correctness fixes - the day was punctuated by Patrick Steinhardt's satirical MD5 hash algorithm proposal that sparked playful responses from multiple contributors. The maintainer Junio Hamano kept the tone light with his characteristic winking emoji response to the joke patch.

### Notable Threads

**Fsmonitor split-index interaction fixes**  
Paul Tarjan provided detailed technical justification for the fsmonitor split-index interaction fixes in response to Junio Hamano's concerns. The response clarified that bitmap positions are written against the full index but read before merge, making out-of-bounds positions legitimate during the read phase. The discussion shows the series (now at v12) addressing even edge cases with thorough consideration of the index lifecycle and existing fallback mechanisms.

**Xdiff regression in merged series**  
Phillip Wood identified a regression in the recently merged xdiff refactoring series that affects diff output formatting. The issue stems from patch 6/6 modifying action arrays during INVESTIGATE handling, which wasn't caught during pre-merge testing. This demonstrates how subtle issues can surface even after thorough review, with Phillip suggesting using local variables to preserve the original behavior while maintaining readability improvements.

**Remote group push failure isolation**  
Junio Hamano provided concrete implementation guidance for Usman Akinyemi's remote group push feature, outlining two process spawning strategies to achieve the agreed-upon failure isolation semantics. The discussion has progressed from architectural decisions to final implementation details, suggesting the feature is nearing readiness for merging with only this engineering question remaining.

**Graph lane limit display glitch**  
Johannes Sixt reported a display glitch in Pablo Sabater's `--graph-lane-limit` feature where truncation markers ('~') appear misaligned in merge-heavy histories. The author acknowledged the issue and proposed exploring a fundamental redesign using gitk-style column rearrangement, potentially delaying the current series in favor of more comprehensive improvements to graph rendering.

**ODB write interface completion**  
Justin Tobler's series completing the ODB write operations interface received positive review from Patrick Steinhardt, with only minor nits remaining. The patches establish a pluggable write interface that enables alternative storage backends while maintaining existing packfile behavior, marking a key milestone in the ODB abstraction effort.

### In Brief

**Reftable portability discussions** -- Patrick Steinhardt addressed design questions about fsync behavior and header organization in the reftable portability series, while defending the `REFTABLE_INLINE` macro's necessity for libgit2's C90 compatibility needs.

**Backfill sparse-checkout fix** -- Trieu Huynh clarified their patch making `git backfill` respect sparse-checkout configurations, explaining how the -1 sentinel allows falling back to config values when no explicit flag is given.

**Commit-a crash fix** -- Nick Golden submitted v2 fix for a `git commit -a` crash during unresolved merges, incorporating Junio's test improvement suggestions by replacing Perl with sed and simplifying assertions.

**C23 const-correctness reviews** -- Jeff King's series received detailed feedback on the `skip_prefix()` macro solution, with Phillip Wood initially concerned about behavior changes before retracting his objection after realizing the dead branch wouldn't affect the contract.

**IncludeIf worktree conditions** -- Chen Linxuan introduced a well-received series adding `worktree` and `worktree/i` conditions for config includes, solving multi-worktree path matching pain points with thorough documentation and tests.

### On the Radar

**Dash shell test fixes** -- Patrick Steinhardt's series addressing test failures with Dash v0.5.13 may need additional documentation (per Eric Sunshine's suggestion) to prevent accidental reversion of the heredoc workarounds.

**Promisor remote config split** -- Christian Couder plans to restructure his promisor remote series into preparatory and functional parts based on Patrick Steinhardt's security-focused review feedback.

**Git Rev News published** -- The 133rd edition summarizing March's Git development activity is now available, with issue #842 open for contributions to the next installment.