Here's the daily digest for April 1, 2026:

**The day in brief.** April 1st saw a mix of serious technical discussions and lighthearted April Fools' humor across 74 emails in 28 threads. Key developments include progress on the fsmonitor daemon's split-index handling, a regression fix in xdiff refactoring, and final reviews for several features nearing completion. The day also featured Git's traditional April 1st joke patches and discussions.

**Notable threads**

**Fsmonitor split-index interaction fixes**  
Paul Tarjan provided detailed technical justification for the fsmonitor daemon's split-index handling, addressing Junio Hamano's concerns about assertion removal safety. The response clarified that bitmap positions are written against the full merged index but read before merge, making out-of-bounds positions legitimate during the read phase. The series remains technically complete at v12, with all implementation requirements met and production validation.

**xdiff regression from refactoring**  
Phillip Wood identified a regression in the recently merged xdiff refactoring series that affects git log output formatting. The issue stems from patch 6/6 modifying action arrays during INVESTIGATE handling, impacting subsequent xdl_clean_mmatch() calls. Phillip suggested using local variables instead to preserve the original behavior while maintaining readability improvements. This post-merge review shows how subtle issues can surface even after thorough pre-merge testing.

**Remote group push failure isolation**  
Junio Hamano provided concrete implementation strategies for failure handling in Usman Akinyemi's remote group push feature. He outlined two approaches for process isolation - spawning N children or N-1 children plus parent - while maintaining the agreed-upon ";" semantics for failure handling. The series is in its final implementation phase after architectural decisions were settled in previous discussions.

**Graph lane limit display glitch**  
Johannes Sixt reported a display glitch in Pablo Sabater's `--graph-lane-limit` feature where truncation marker ('~') alignment fails in merge-heavy histories. Sabater confirmed the issue and proposed exploring gitk-style column rearrangement as an alternative solution, potentially delaying the current truncation approach. Junio later expanded the discussion to include long-standing root commit spacing issues in graph rendering.

**In brief**

**Reftable portability** -- Patrick Steinhardt defended the `REFTABLE_INLINE` macro abstraction, citing libgit2's C90 compatibility needs despite universal C99 support.

**ODB write operations** -- Justin Tobler's series completing the object database transaction interface received final review acks from Patrick Steinhardt, with minor nits remaining.

**Mailmap toggle docs** -- Jean-Noël Avila confirmed proper AsciiDoc syntax for Siddharth Asthana's already-merged mailmap toggle feature.

**Backfill sparse-checkout** -- Trieu Huynh clarified their patch makes `git backfill` respect core.sparseCheckout config by using -1 sentinel value.

**Commit -a crash fix** -- Nick Golden submitted v2 of a fix for `git commit -a` crashes during unresolved merges, incorporating test improvements from Junio's review.

**On the radar**

**C23 const-correctness** -- Jeff King's comprehensive series addressing const-preserving fixes saw extensive review, with maintainer approval on several patches but ongoing discussion about macro naming patterns.

**Promisor remote config** -- Christian Couder plans to split his series into preparatory refactoring and feature patches based on Patrick Steinhardt's security-focused feedback.

**Test modernization** -- Zakariyah Ali's GSoC microproject to modernize t2000 tests received final review feedback from Tian Yuchen about test helper usage.

**April Fools' humor**  
The list saw its traditional April 1st levity with Patrick Steinhardt's well-crafted MD5 hash algorithm proposal, sparking a thread of increasingly absurd suggestions culminating in Tian Yuchen's carrier pigeon transport scheme. Junio acknowledged the jokes with characteristic restraint (a single winking emoji).