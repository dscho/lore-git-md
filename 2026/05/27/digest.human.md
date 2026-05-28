# Here's the Git mailing list daily digest for May 27, 2026:

---

### The day in brief
A busy Wednesday with 68 emails across 16 threads, featuring significant performance optimizations, infrastructure improvements, and several patch series reaching maturity. Key highlights include Taylor Blau's bitmap generation speedups, Christian Couder's promisor remote enhancements ready for merging, and Kristofer Karlsson's revision walking optimizations. Junio also released the monthly "What's cooking" report tracking project progress.

---

### Notable threads

### Performance optimizations for bitmap generation

Taylor Blau's 8-part series optimizing pack-bitmap-write received thorough review from Jeff King, demonstrating dramatic improvements: 60% faster generation (612s->294s) and 72% smaller bitmaps (635MB->176MB) in test repositories. The changes systematically eliminate bottlenecks through position caching, commit sorting, and restructuring pseudo-merge handling. Peff validated each optimization's technical merits while questioning some counterintuitive speedups. The series is now ready for integration after addressing all feedback.

### Promisor remote auto-completion ready

Christian Couder's URL-based promisor remote configuration series (v4) completed final documentation refinements and is now merge-ready. The work allows automatic remote configuration based on URL patterns with strict security constraints, building on the previously approved LOP series. Changes since v3 were minimal - renaming a helper function and fixing documentation formatting. Patrick Steinhardt, Toon Claes, and Junio have all signed off after multiple review rounds.

### Revision walking with priority queues

Kristofer Karlsson proposed a 3-patch series optimizing revision walking by replacing O(w) insertion costs with priority queue operations. The changes show 5.3x speedups for merge-heavy repositories by reducing queue maintenance from 79% to 16% of runtime. The careful approach first fixes a memory leak, then refactors walk mode dispatch before implementing the queue optimization. Benchmarks validate significant wins while maintaining compatibility and showing no regression for narrow frontiers.

### Line-log integration with standard diff pipeline

Michael Montalbo's series unifying line-log output with Git's standard pipeline received final approval from D. Ben Knoble, with only a test helper documentation nit remaining. The changes fix inconsistencies where `-L` bypassed features like pickaxe and diff filters while aligning commit/diff separator behavior. The v2 iteration incorporated all substantive feedback and is now ready for merging.

### Git daemon network fixes

Sebastien Tardif's bugfix series for IPv6 and NULL pointer issues in `daemon.c` reached v2 with all maintainer feedback addressed. The patches fix address corruption, IPv6 truncation, and REMOTE_PORT logging safety. Junio provided final review focusing on commit message wording for the NULL check patch, acknowledging its defensive value despite technical redundancy. The series appears ready for merging after thorough platform-specific validation.

---

### In brief

**`git branch --contains` optimization** -- Kristofer Karlsson proposed enabling cached DFS algorithm for branch operations when commit-graph exists, showing 3.7-16x speedups in benchmarks.

**Shell completion dotfile behavior** -- Zakariyah Ali's patch to hide dotfiles by default in path completion received conceptual approval from Junio, pending technical review from shell experts.

**Deprecated function removal** -- Kristoffer Haugsbakk cleaned up commit.h by removing functions marked for deprecation since Git 2.53.0 after verifying no remaining uses.

**Commit-reach.c refactoring** -- René Scharfe removed redundant sorting in `paint_down_to_common()`, eliminating unnecessary work since callers either re-sort or ignore order.

**MacOS linker warning fix** -- A small patch initializes `packet_buffer` in pkt-line.c to avoid alignment warnings during linking on macOS.

**Fetch connectivity optimization withdrawn** -- Kristofer Karlsson retracted a proposed fetch optimization after Jeff King identified it wouldn't activate as intended outside clone operations.

**`git mv --index-only` discussion concludes** -- The thread reached consensus that while technically feasible, the feature doesn't currently warrant core implementation given available workarounds.

**Word-diff alignment RFC** -- A prototype `--word-diff-align` option was proposed to highlight edits within moved blocks, with initial test case demonstrating permission bitmask changes.

---

### On the radar

**Linux fsmonitor** -- Paul Tarjan's implementation appears ready for merging per Junio's "What's cooking" report, bringing Linux to parity with existing Windows/macOS backends.

**ODB abstraction** -- Patrick Steinhardt's object database rework continues progressing, with transaction interface changes recently graduated to 'next'.

**Documentation modernization** -- Jean-Noël Avila's synopsis-style conversion effort remains active, with several patches in flight.

**Sparse index optimizations** -- Derrick Stolee's work on `git restore` with sparse index was noted as cooking in Junio's report.