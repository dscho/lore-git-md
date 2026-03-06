# Git Mailing List Digest - 2025/10/17

**The day in brief.** A busy Friday with 76 emails across 22 threads saw significant progress on several fronts: Adrian Ratiu's hook subsystem refactoring reached v2, Taylor Blau's massive repack series concluded review, and multiple documentation improvements landed. Security updates dominated the news with Git for Windows 2.51.1 addressing CVE in Git LFS. Notable threads included a regression in `git diff --quiet` behavior and ongoing discussions about SSH agent test infrastructure.

## Notable threads

### Hook subsystem refactoring reaches v2

Adrian Ratiu and Emily Shaffer's 10-part series converting Git's hooks to use a unified `hook.h` API reached its second iteration today. The comprehensive refactoring introduces stdin handling for parallel processes (`feed_pipe_fn`), output capture capabilities (`consume_sideband_fn`), and batched processing while maintaining backward compatibility. Key changes in v2 include stricter `BUG()` enforcement for mutually exclusive stdin methods, simplified post-rewrite and pre-push callbacks with strbuf reuse, and removal of problematic keepalive logic in update hooks. The series now appears ready for final review pending resolution of minor test failures in patch 6's sideband handling.

### Repack refactoring concludes review

Taylor Blau's 49-part series refactoring Git's repack machinery to support incremental MIDX-based repacking while removing global variables reached consensus today. Reviewers (Jeff King, Patrick Steinhardt, and Junio Hamano) confirmed all v1 feedback has been addressed in v2, including const-correctness improvements, proper bool return types, and header cleanup. The changes successfully modularize the repack machinery while preserving legacy behavior, setting the stage for future MIDX/bitmap work. Several follow-up opportunities were noted (marked as #leftoverbits) but deemed non-blocking for this substantial architectural cleanup.

### `git diff --quiet` regression identified and fixed

Jake Zimmerman reported and Jeff King quickly diagnosed a regression in Git v2.51.1 where `git diff --quiet HEAD` incorrectly produces output for staged files when it should remain silent. The bisect pointed to commit b55e6d36eb ("diff: ensure consistent diff behavior with ignore options") as the culprit. Jeff proposed a conservative fix redirecting output to /dev/null in quiet mode, which Junio Hamano endorsed as a "stupid but cannot be incorrect" solution. The thread revealed gaps in test coverage for porcelain diff's quiet behavior that allowed this regression to slip through.

### SSH agent test infrastructure issues

Multiple contributors reported and analyzed test failures in `t7528-signed-commit-ssh.sh` related to SSH agent socket path handling. The discussion uncovered two distinct problems: path length limitations causing ENAMETOOLONG errors, and a more serious security issue where unquoted path evaluation could potentially execute arbitrary commands. Jeff King traced the latter to OpenSSH's version-dependent socket location behavior (pre-10.1 using /tmp vs post-10.1 using $HOME). The thread is converging on solutions that handle both old and new OpenSSH versions while addressing the security implications of the current unquoted eval approach.

## In brief

**Promisor pack object handling bug** -- Sam Estep provided a reliable reproduction case for intermittent BUG() assertions when processing non-existent objects during promisor pack operations, moving this long-running issue closer to resolution.

**Documentation formatting fixes** -- Kristoffer Haugsbakk and Julia Evans collaborated on post-merge improvements to `git-checkout` man page formatting, correcting AsciiDoc markup for consistent rendering.

**gitk window geometry persistence** -- Johannes Sixt finalized a two-part series restoring full window geometry saving in gitk, with Mark Levedahl confirming the updated commit message properly documents the historical multi-monitor concerns.

**Commit-graph segfault with alternates** -- Philipp Hahn reported a segmentation fault when using commit-graphs with repositories that have alternates configured, occurring when writing tree hashes from a NULL pointer.

**CR/LF handling in GPG verification** -- Christian Couder provided detailed feedback on Okhuomon Ajayi's bugfix patch, focusing on commit message improvements to better explain the behavior change.

**Dry-run diff output fixes** -- Lidong Yan's patch to complete dry-run output suppression was approved by Johannes Schindelin for Git for Windows, though Junio Hamano suggested it may target v2.51.2 instead.

**Git for Windows security release** -- Version 2.51.1 was announced, addressing high-severity CVE-2025-26625 in Git LFS and marking git svn for removal in the next major release.

**git-reset documentation overhaul** -- Julia Evans completed a 4-part series reorganizing the man page to better match user workflows, with Junio Hamano providing final wording refinements.

## On the radar

**Protocol.file.allow configuration** -- A reported inconsistency in local vs global configuration handling for submodule operations remains unresolved, with Jeff King explaining the architectural constraints that make this a known limitation.

**git last-modified optimizations** -- Toon Claes' performance work using path-tracking bitmaps continues to evolve, with recent discussion focusing on memory management optimizations and correctness verification against `git log --max-count=1`.

**Geometric repack strategy** -- Patrick Steinhardt and Taylor Blau discussed future enhancements to the new maintenance strategy, including potential configuration options for the geometric split factor and optimizations for reachability checking.