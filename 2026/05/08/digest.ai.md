Here's the daily digest for May 8, 2026:

**The day in brief.** A moderately busy Friday with 54 emails across 18 threads, featuring several significant patch series reaching completion. Key developments include Johannes Schindelin's Windows platform fixes for large objects and nedmalloc removal, Kristoffer Haugsbakk's finalized git-interpret-trailers documentation, and substantive discussions about merge handling in git history.

**Notable threads**

**Windows platform improvements finalized**  
Johannes Schindelin's two major Windows-focused series reached completion today. The 11-patch large object handling series (now v3) systematically addresses 32-bit truncation issues during cloning operations, with new test infrastructure and CI optimizations. His 6-patch nedmalloc removal series (v3) completes the deprecation of this unmaintained allocator, with all technical concerns resolved and only integration mechanics remaining. Both series have received explicit approvals from key reviewers (Patrick Steinhardt and Derrick Stolee).

**git-interpret-trailers documentation complete**  
Kristoffer Haugsbakk's 9-patch documentation series for git-interpret-trailers has been merged after addressing all review comments. The series clarifies behavior around trailer key formats (ASCII alphanumeric + hyphen only), standardizes terminology ("metadata", "trailer block"), and adds concrete examples of both valid usage and failure cases. The work originated from a bug report about period-containing keys being rejected and evolved into comprehensive documentation improvements.

**Merge handling in git history**  
Phillip Wood provided substantive review of Johannes Schindelin's RFC series adding merge commit support to `git history`. The discussion focuses on edge cases in conflict resolution preservation, particularly when rewritten parents introduce new conflicts not present in the original merge. Phillip proposed alternative approaches for single-parent rebase cases, prompting ongoing technical discussion about the tradeoffs between implementation complexity and behavior correctness.

**In brief**  

**Test pollution fix** -- Junio C Hamano corrected a long-standing test pollution issue in t5551 HTTP fetch tests that only surfaced with GIT_TEST_LONG enabled, with Jeff King confirming the fix as "obviously correct".

**git-jump usability** -- Greg Hurrell's patch adding automatic mode selection to the contrib script received approval after discussion about UX tradeoffs, with Jeff King acknowledging the typing efficiency benefits outweigh initial discoverability concerns.

**Promisor remote security** -- Toon Claes reviewed Christian Couder's URL-based promisor remote configuration, verifying security mechanisms for URL pattern handling and test consistency.

**rebase --update-refs fix** -- A bugfix for incorrect ref updates when rebase.instructionFormat shows decorations reached v2 with Phillip Wood's approval, now with simplified test verification.

**On the radar**  

**Maintenance subsystem resource usage** -- Jeff King's analysis of uncontrolled resource consumption during `git add`/`commit` has identified maintenance.auto as the root cause rather than gc.auto, with potential repository corruption from concurrent repacks now awaiting Derrick Stolee's input.