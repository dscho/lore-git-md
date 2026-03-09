Here's the Git mailing list digest for January 12, 2026:

**The day in brief.** A busy Monday with 145 emails across 26 threads, featuring significant progress on multiple fronts. The standout developments include final approval for the submodule gitdir path encoding series and the `git-history` command infrastructure, plus ongoing work on ODB abstraction and `the_repository` removal. Several test modernization and documentation efforts also moved forward.

**Notable threads**

**Submodule gitdir path encoding reaches completion**  
Adrian Ratiu's long-running series to implement runtime configuration for submodule gitdir paths via `extensions.submodulePathConfig` has received final approval from Junio Hamano after nine iterations. The v9 patch set addresses Patrick Steinhardt's last polish comments with minor cleanups to test configuration and message formatting. The implementation provides a four-tier fallback system (plain → encoded → numbered → hashed paths) with comprehensive case-folding protection and test coverage. Josh Steadmon confirmed successful testing against Google's internal use cases, providing real-world validation. With all technical concerns resolved, this marks the conclusion of a multi-month effort to safely handle submodule paths across different filesystems.

**git-history command infrastructure nears merge**  
Patrick Steinhardt's series introducing the experimental `git-history` command has addressed all substantive technical feedback and appears ready for merging pending resolution of SZEDER Gábor's forward-compatibility concerns. The v10 patches refine the replay infrastructure with Elijah Newren's improvements to detached HEAD handling and interface simplification. While the current subcommands (`reword` and `split`) are conflict-free by design, SZEDER raised questions about how future conflict-prone operations would integrate. Patrick argues these can be addressed separately, suggesting the current functionality provides immediate value. Junio provided final polish on boolean logic style, indicating the series is in its final stages.

**ODB abstraction progresses with object info refactoring**  
Patrick Steinhardt's 7-patch series refactoring object info handling as part of the ODB abstraction effort has been approved by Karthik Nayak (GitLab ODB team). The changes introduce precise delta type tracking, simplify interfaces, and optimize disk size lookups (9% speedup in `git rev-list --disk-usage`). The v5 iteration adds a regression test for a segfault in `git archive` when handling deltified blobs exceeding core.bigFileThreshold, which the series incidentally fixes. This represents another completed step in the multi-year effort to make Git's object storage layer more flexible and maintainable.

**the_repository removal advances for tree parsing**  
René Scharfe's 10-patch series to eliminate `the_repository` dependencies in tree parsing has reached its final discussion phase. The work introduces repository-aware variants of tree parsing functions and systematically converts callers across multiple subsystems. The only remaining question concerns how to document the deprecation timeline for compatibility wrappers, with Junio Hamano reaffirming the project's policy against version-specific deprecation markers. Patrick Steinhardt suggested alternative phrasing that records intent without creating artificial deadlines, which appears to be converging toward consensus. This marks significant progress in the long-running effort to remove global state from Git's internals.

**In brief**  

**HTTP authentication fix** -- Aaron Plattner and Patrick Steinhardt refined a fix for HTTP authentication in remote-curl when handling large requests with probe_rpc(), particularly affecting filtered clones. The solution ensures credentials are included in probe_rpc() calls.

**repo info --keys finalized** -- Lucas Seiki Oshiro's series adding `--keys` to `git repo info` received final review from Patrick Steinhardt, who suggested minor documentation clarifications about format naming.

**Windows symlink prep approved** -- Johannes Schindelin and Karsten Blees' Windows symlink series addressing mingw compatibility and config parsing received Patrick Steinhardt's ack for merging.

**status tracking enhancement ready** -- Harald Nordgren's 23-iteration series to show both upstream and push tracking in `git status` output is now approved for `next` after addressing Phillip Wood's feedback on advice display consistency.

**test modernization** -- Deveshi Dwivedi's series modernizing post-checkout hook tests in t5403 was merged, introducing a `check_post_checkout` helper and standardizing validation patterns.

**On the radar**  

**Geometric repacking with promisors** -- Patrick Steinhardt's series addressing incompatibility between geometric repacking and promisor remotes continues discussion about edge cases in promisor pack handling, with Taylor Blau providing review feedback.