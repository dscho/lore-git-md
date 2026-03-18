Here's the Git mailing list digest for March 17, 2026:

## The day in brief
A busy day with 89 emails across 38 threads, featuring significant progress on several fronts. Key highlights include final refinements to the `git replay --revert` feature, HTTP 429 rate limit handling improvements, and multiple GSoC proposal discussions. Junio's "What's cooking" report shows several important topics graduating to 'master', while Derrick Stolee's `git backfill` enhancements promise better partial clone support.

## Notable threads

**HTTP 429 rate limit retry support finalized**  
Vaidas Pilkauskas's v6 series implementing HTTP 429 (Too Many Requests) handling is now technically complete after addressing Jeff King's architectural feedback. The implementation adds configurable retry behavior with `http.maxRetries`, `http.retryAfter`, and `http.maxRetryTime` settings, respecting server-specified Retry-After headers. All preparatory strbuf fixes have been merged separately, leaving just the core HTTP functionality for final review.

**`git replay` root commit handling**

A new patch enables `git replay` to process repository root commits (those with no parents) by properly handling empty tree cases and removing previous limitations. Junio-approved changes in `replay.c` now allow replaying full commit history back to the initial commit. The implementation maintains consistency with non-root cases while adding test coverage for this previously unsupported scenario.

**`git checkout -m` autostash improvements reach v6**

Harald Nordgren's series teaching `git checkout -m` to use autostash when switching with conflicts has incorporated final error handling refinements. The implementation now includes proper lock file cleanup and builds on earlier stash labeling improvements. With all substantive feedback addressed and 18 test cases in t7201-co.sh, the series appears ready for merging pending final documentation polish.

**Reference transaction hook gains "preparing" phase**

Eric Ju's v3 patch adding a "preparing" phase to reference-transaction hooks has addressed Junio's final suggestions about error message formatting. The change helps distributed Git systems like GitLab's Gitaly coordinate concurrent writes before locks are acquired. The implementation maintains backward compatibility while providing clearer error messages that identify both hook and phase.

## In brief

**`git backfill` enhancements** -- Derrick Stolee's 5-part series adds revision walking and pathspec support to `git backfill`, enabling targeted blob downloads in partial clones. The implementation shows careful attention to performance with prefix optimizations.

**Line-log diff improvements** -- Michael Montalbo's completed series routing `git log -L` through the standard diff pipeline now has final documentation formatting fixes from Kristoffer Haugsbakk.

**Subcommand autocorrection** -- Jiamu Sun's parse-options API extension for subcommand autocorrection awaits final merge after addressing Junio's questions about edit distance thresholds.

**Worktree current-ness semantics** -- Phillip Wood provides additional justification for changing `is_current_worktree()` behavior as part of `the_repository` removal, citing ref store efficiency benefits.

**Commit-graph generation fix** -- Patrick Steinhardt corrects a regression in generation number calculations for commits with dates exceeding 34 bits (post-year 2514).

**Whitespace patch application** -- Junio fixes an edge case where empty context lines in patches could trigger incorrect whitespace errors during application.

## On the radar

**Exit code standardization** -- Emerging consensus suggests changing `--help` to exit 0 across Git, but the discussion reveals this would be a widespread behavioral change affecting many commands.

**GSoC proposal refinement** -- Multiple students are iterating on proposals for `git repo` enhancements, path handling improvements, and partial clone optimizations with mentor feedback.

**`the_repository` removal** -- Coordination continues around parallel efforts to eliminate global state, with Junio noting integration considerations between Shreyansh Paliwal's and Patrick Steinhardt's changes.