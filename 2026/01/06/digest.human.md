Here's the Git mailing list daily digest for January 6, 2026:

**The day in brief.** A busy day with 79 emails across 27 threads, featuring significant progress on multiple fronts. Key developments include final reviews for the submodule gitdir path encoding series, resolution of branch selection strategy for `git-history`, and performance investigations into `pack-refs` behavior. Several bugfix series reached merge-ready status while UI improvements for `git add -p` continued refinement.

**Notable threads**

**Submodule gitdir path encoding reaches final polish**  
Patrick Steinhardt's long-running series to make submodule gitdir paths configurable at runtime has reached its final review stage. The v7 implementation of `extensions.submodulePathConfig` received detailed feedback from Patrick himself, focusing on minor documentation and test improvements. With Junio Hamano's earlier ack and all architectural concerns resolved, this series appears ready for merging after addressing these last nits. The changes enable runtime configuration of submodule paths through `submodule.<name>.gitdir` with comprehensive handling of edge cases like nested submodules and case-insensitive filesystems.

**git-history branch selection strategy settled**  
The contentious discussion around `git-history`'s branch rewrite scope reached resolution, with Patrick Steinhardt accepting Elijah Newren's proposal to limit rewrites to local branches (refs/heads/*) only. This addresses concerns about rewriting shared history while maintaining the command's utility for local workflow. Patrick also incorporated Phillip Wood's performance analysis showing that repository ref distributions make the branch walking approach practical. The series now moves forward with clearer boundaries and documented expectations.

**Memory management fixes ready for merging**  
Patrick Steinhardt's bugfix series addressing memory leaks in commit-graph writing and submodule ODB handling received final review acks from Karthik Nayak and Justin Tobler. The two-patch series fixes distinct issues: a commit-graph memory leak during merge commit processing and a submodule ODB cleanup sequence that leaked 192 bytes per access. With all feedback addressed and test coverage validated, these focused fixes are poised for inclusion.

**In brief**  

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**git add -p UI improvements** -- Abraham Samuel Adekunle's GSoC project adds visual feedback for previous hunk decisions, now in its sixth iteration with ongoing discussion about optimal prompt formatting.

**Performance test fixes** -- Jeff King corrects two issues in the performance test infrastructure affecting output directory handling and environment variable preservation.

**pack-refs performance investigation** -- Analysis of `pack-refs` slowness on NFS shifts focus to mmap access patterns, with tests showing pre-reading pack files dramatically reduces runtime.

**On the radar**  

**git diff --relative behavior** -- Emerging discussion about whether `--relative` should treat its argument as a text prefix or directory, with Johannes Sixt demonstrating problematic edge cases.

**Bazel build support** -- Son Luong Ngoc scales back in-tree Bazel support plans after technical objections, now pursuing an out-of-tree solution via Bazel Central Registry.