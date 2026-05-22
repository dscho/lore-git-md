Here's the daily digest for 2026/05/21:

---

### The day in brief

Thursday, May 21, 2026 saw moderate activity with 103 emails across 23 threads. The day was dominated by technical refinements to several major features nearing completion, including date parsing fixes, branch pruning improvements, and the ongoing ODB abstraction work. Notable progress included Junio's approval of Tuomas Ahola's approxidate series and resolution of the `--track=fetch` design question.

---

### Notable threads

**Date parsing edge cases resolved**  
Tuomas Ahola's five-patch series fixing edge cases in Git's approxidate parser reached its final form with Junio C Hamano's approval. The changes correct handling of special time formats ("noon", "tea") when combined with explicit date references, resolving a longstanding quirk dating back to 2006. The v5 iteration added comprehensive test case for "January 5th today pm" to verify the new "today=midnight" semantics. With all feedback addressed and Junio indicating he'll queue the series, this represents the successful conclusion of a careful refinement process.

**Branch pruning interface finalized**  
Harald Nordgren's `git branch --prune-merged` series reached v10 with a revised interface design based on maintainer feedback. The implementation now supports both remote-tracking and local upstream branches via flexible branch/glob syntax while retaining all safety checks. Junio C Hamano had strongly endorsed Phillip Wood's suggestion to generalize beyond remote-specific pruning, resulting in a more versatile command that accepts literal refs, local branches, and shell-style globs. The addition of `--dry-run` mode completes the feature's evolution through ten iterations.

**Checkout's new --track=fetch mode**  
After twelve iterations, Harald Nordgren's `git checkout/switch --track=fetch` feature patch resolved its final architectural question about handling ambiguous remote-tracking refs. Junio C Hamano and Phillip Wood converged on reusing existing tracking branch resolution behavior from `branch.c` rather than inventing new logic. The feature automatically fetches when creating tracking branches, eliminating a common workflow pain point. With comprehensive test coverage (18 cases) and documentation complete, the series appears ready for merging pending final maintainer approval.

**External notes command debate**  
Siddh Raman Pant's series adding external command support for Git notes generated substantive discussion about both technical design and contribution policies. Johannes Sixt objected to core Git implementing timeout handling for helper processes, arguing this indicated protocol flaws, while Oswald Buddenhagen countered that timeouts are essential for robustness. Separately, brian m. carlson raised concerns about AI-assisted contributions conflicting with Git's stricter policies compared to Linux. The thread now has three parallel tracks: timeout design (technical), AI policy (process), and code style (implementation).

**Loose object refactoring advances**  
Patrick Steinhardt continued his ODB abstraction work with an 18-patch series converting loose object handling to callback-based operations. The mechanical changes move functionality from object-file.c to odb/source-loose.c while maintaining all existing behavior. Junio C Hamano provided light review, catching minor documentation nits but not engaging with the substantive changes. The series systematically wires up each odb_source callback, with transaction support explicitly deferred for future work. This represents another incremental step in the long-term effort to make Git's object storage pluggable.

---

### In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**GitLab CI upgrades** -- Patrick Steinhardt updates macOS runners to M2 Pro hardware and newer base images, cutting test times by 30-40%.

**Documentation polish** -- Kristoffer Haugsbakk sent two series (9 patches total) improving hook and replay command documentation through better cross-referencing and formatting.

**Maintenance bugfix backport** -- Patrick Steinhardt's fixes for detached mode lockfiles and gc.auto behavior will be included in the upcoming 2.54.1 release.

**On the radar**

**Rustification tensions** -- While not discussed today, Ezekiel Newren's Rust integration work remains a potential flashpoint given Randall Becker's ongoing concerns about NonStop platform support.