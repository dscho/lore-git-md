# Git Mailing List Digest - 2026/05/27 (Wednesday)

**The day in brief.** A moderately busy Wednesday with 68 emails across 16 threads, featuring significant progress on performance optimizations, promisor remote configuration, and several cleanups. Key highlights include Taylor Blau's bitmap generation optimizations receiving final approval, Christian Couder's promisor remote series reaching merge readiness, and Kristofer Karlsson's revision walking priority queue series showing dramatic speedups for large repositories.

## Notable threads

### **Bitmap generation optimizations complete with 60% speedup**

Taylor Blau's 8-patch series optimizing pack-bitmap-write has concluded review with Jeff King's (Peff) final approval. The changes systematically improve bitmap generation performance for large repositories, achieving a 60.1% speedup (612.5s->294.1s) and 72% storage reduction (635MB->176MB) in test cases. The series culminates with a restructuring of pseudo-merge handling to build after regular bitmaps, reducing their overhead from +95.5% to just +11.7%. Peff's thorough review validated both the technical approach and real-world performance gains, noting the work "looks correct" while acknowledging some counterintuitive optimizations that proved effective. The changes maintain backward compatibility while significantly improving large repository performance.

### **Promisor remote auto-configuration ready for merging**  

Christian Couder's v4 series implementing URL-based auto-configuration for promisor remotes is now technically complete and ready for merging. The work builds on the previously approved LOP (Large Object Promisors) series, adding `promisor.acceptFromServerUrl` configuration with strict security constraints. Key features include URL pattern matching with glob support, auto-generated remote names (`promisor-auto-` prefix), and collision handling to prevent name hijacking. The final version incorporates all review feedback including naming consistency fixes from Junio and documentation improvements from Kristoffer Haugsbakk. The series represents significant progress in making promisor remotes more configurable while maintaining strong security defaults.

### **Revision walking gets priority queue optimization**

Kristofer Karlsson introduced a 3-patch series optimizing revision walking by replacing O(w) insertion costs with priority queue operations in `get_revision_1()`. The changes show dramatic improvements for merge-heavy repositories (5.3x faster for a 2.4M commit monorepo) while having no measurable impact on repositories with narrow walk frontiers. The series takes a careful approach - first fixing a memory leak, then introducing a `rev_walk_mode` enum to clean up dispatch logic, and finally converting the streaming walk to use a priority queue. Benchmark data shows queue maintenance dropping from 79% to 16% of runtime in target cases. The work follows the successful pattern of an earlier `limit_list()` optimization and appears well-positioned for integration.

### **Line-log integration with standard diff pipeline approved**

Michael Montalbo's v2 series integrating line-log output with Git's standard diff pipeline received final approval from D. Ben Knoble, with only a minor documentation nit about test helper usage remaining. The changes unify line-log behavior with `log_tree_diff()` to support features like pickaxe (`-S`) and `--diff-filter` that were previously bypassed. The implementation routes line-log through the diff pipeline while maintaining stat format restrictions as follow-up work. Knoble's review confirms the series achieves its goal of consistent diff output handling, with all substantive feedback from v1 addressed in this iteration.

## In brief

**`git branch --contains` optimization proposal** -- Kristofer Karlsson revives discussion with benchmark data showing 3.7-16x speedups from enabling cached DFS algorithm when generation numbers are available via commit-graph.

**Git daemon network fixes** -- Sebastien Tardif's v2 series addresses IPv6 support issues and NULL pointer safety in daemon mode, with Junio providing final wording feedback on the NULL check rationale.

**Shell completion dotfile behavior** -- Junio conceptually approves Zakariyah Ali's patch to hide dotfiles by default in path completion, deferring technical review to shell experts.

**Fetch connectivity check optimization withdrawn** -- Kristofer Karlsson retracts proposal after Jeff King identified the `self_contained_and_connected` optimization cannot activate during fetches as originally claimed.

**`git mv --index-only` discussion concludes** -- Junio confirms technical soundness but downplays urgency, effectively tabling the feature unless volunteer implementers emerge.

**New `--word-diff-align` RFC** -- Initial proposal for highlighting edits within moved code blocks, with prototype demonstrating alignment of permission bitmask changes (0x0008->0x0001) in moved C header blocks.

**Deprecated commit list functions removed** -- Kristoffer Haugsbakk completes cleanup removing `free_commit_list` and related functions marked for deprecation since Git 2.53.0.

**Redundant sort in commit reachability** -- René Scharfe removes unnecessary date-based sorting in `paint_down_to_common()` that callers either re-sort or ignore.

**macOS linker warning fix** -- Initializes `packet_buffer` in pkt-line.c to avoid alignment-related warnings during linking on macOS.

**Path-walk with delta-islands** -- Taylor Blau's 3-patch series enables `--walk-path` to work with both reachability bitmaps and delta-islands, completing integration started by Derrick Stolee.

## On the radar

**Linux fsmonitor implementation** -- Flagged as ready for merge in Junio's "What's cooking" report after extensive development by Paul Tarjan.

**ODB transaction interface** -- Significant rework enabling in-memory object storage continues progressing through integration branches.

**`the_repository` removal** -- Ongoing effort sees continued patches moving functions away from the global variable.