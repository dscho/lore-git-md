# Git Mailing List Digest - 2026/03/31

**The day in brief.** A busy day with 161 emails across 37 threads, featuring significant progress on multiple fronts. Key developments include final approvals for the fsck refactoring and ODB abstraction series, ongoing discussions about AI-assisted translation workflows, and several performance optimizations for promisor packs and MIDX handling. Junio Hamano's "What's cooking" report provides a comprehensive overview of the current integration state.

## Notable threads

**Linux fsmonitor implementation reaches final polish**  
The long-running series adding Linux inotify support to fsmonitor is in its final stages, with Paul Tarjan addressing the last known edge case involving split-index interactions. Junio Hamano raised important questions about the fix's scope, particularly whether removing assertions in non-split-index cases might mask real bugs. The discussion highlights Git's careful approach to maintaining bisectability even when fixing edge cases. With production validation from multiple months and all substantive technical feedback addressed, the series appears technically complete pending resolution of these final process considerations.

**Reftable portability series for libgit2 integration**  
Patrick Steinhardt posted a 6-patch series abstracting system dependencies in the reftable code to enable libgit2 integration. The changes introduce platform-agnostic interfaces for inline functions, time handling, memory mapping, and other system calls. Junio Hamano and René Scharfe engaged in detailed discussions about header organization and the `REFTABLE_INLINE` macro's justification. While the technical approach is sound, the thread revealed interesting tensions about how far to go in maintaining compatibility with older compilers versus embracing modern C standards.

**ODB abstraction series approved for merging**  
Patrick Steinhardt's fsck refactoring series to remove `the_repository` dependencies received maintainer approval from Junio Hamano after two rounds of review. The changes move repository state into `struct fsck_options` and introduce proper initialization functions, representing a significant step in Git's architectural cleanup efforts. Separately, Justin Tobler's series adding write operations to the ODB transaction interface concluded with all technical feedback addressed, focusing now on final interface placement details for stream helper functions.

**Configurable submodule fetch errors**  
Paulius Zaleckas introduced a two-patch series making submodule fetch errors configurable when unreachable commits are encountered. The implementation allows choosing between failing (current default) or warning when submodule fetches hit unreachable commits, addressing workflow pain points in environments where missing commits are expected but not required. Jean-Noël Avila provided documentation feedback, noting the need to follow Git's synopsis style for command-line alternatives. The series appears well-structured with comprehensive test coverage.

**Dynamic mailmap toggling in cat-file batch mode**  
Siddharth Asthana's implementation of dynamic mailmap control for `git cat-file --batch-command` reached its final form with maintainer approval. The feature introduces a `mailmap` subcommand with boolean parameters, solving GitLab Gitaly's need to toggle name/email rewriting in long-running processes. Jean-Noël Avila and Junio Hamano contributed final documentation refinements, ensuring the interface is clearly documented following Git's standards. The thread demonstrates successful collaboration between GitLab's production needs and upstream quality standards.

**C23 const-correctness fixes**  
Jeff King (Peff) posted a comprehensive 12-patch series addressing C23 compatibility warnings about const-correctness in `strchr()` usage. The changes span multiple subsystems, with particularly interesting fixes in the pseudo-merge bitmap handling that uncovered deeper issues in on-disk data reading. The series introduces clever macro solutions for type safety in `skip_prefix()` and related functions, demonstrating Peff's characteristic attention to both immediate compiler warnings and long-term code robustness.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**xdiff refactoring complete** -- Ezekiel Newren's series improving type safety and clarity in xdiff's record cleanup logic has been merged to 'next' after addressing all review feedback.

**git replay root commit handling** -- Toon Claes' implementation allowing `git replay` to process root commits is now merged, with ongoing discussion about documentation wording for edge cases.

**Remote group push failure handling** -- Usman Akinyemi and Junio Hamano finalized the semantics for failure propagation when pushing to multiple remotes in a group, opting for "continue after failures" behavior.

**Line-log diff pipeline integration** -- Michael Montalbo's series routing `git log -L` output through the standard diff pipeline is queued for 'next', enabling previously unsupported diff features.

**GSoC proposal submission** -- Abraham Samuel Adekunle's project on prioritized fetching from promisor remotes has been submitted to Google Summer of Code after incorporating Christian Couder's guidance.

## On the radar

**AI-assisted translation workflows** -- Jiang Xin's series remains pending while legal/DCO concerns from brian m. carlson are addressed, though the technical implementation has been production-validated.

**Test infrastructure modernization** -- Patrick Steinhardt takes over maintenance of Junio Hamano's series enabling stricter error detection in the test suite, which faces ongoing shell compatibility challenges.

**git replay --ref validation** -- Toon Claes investigates a SIGABRT in merge-ort when testing invalid refnames, the last blocker for this feature's integration.