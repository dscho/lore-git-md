Here's the Git mailing list digest for March 25, 2026:

**The day in brief.** A busy Thursday with 139 emails across 25 threads, featuring significant progress on several major features including `git replay --revert`, parallel hook execution, and remote group pushing. The day saw multiple patch series reach final polish stages, with maintainer Junio Hamano actively shepherding topics toward integration.

**Notable threads**

**`git replay --revert` reaches v6**  
Siddharth Asthana's series adding revert capability to `git replay` is now in its final iteration, addressing the last compiler warning and documentation nits. The feature enables server-side reverts on bare repositories, primarily for GitLab's Gitaly service, by treating reverts as merges with swapped arguments. After extensive review from Phillip Wood, Patrick Steinhardt, and Junio Hamano across six iterations, the series appears ready for merging with comprehensive test coverage and clear documentation of the horizontal truncation behavior.

**Linux fsmonitor implementation awaits process resolution**  
Junio checks in on Paul Tarjan's Linux fsmonitor series, which is technically complete but has stalled on process questions about submission cadence and AI-assisted development. The v11 implementation addresses all technical feedback including overlayfs reliability and process management, but needs clarification on these procedural points before integration can proceed.

**Parallel hooks configuration merged**  
Adrian Ratiu's parallel hook execution series has been merged to 'next' with post-merge refinements in progress. The implementation establishes comprehensive controls for parallel job counts at global, per-hook, and per-event levels, with safety constraints for non-parallelizable hooks. Final discussions focus on edge cases in the `hook.<event>.enabled` configuration and memory management API design.

**Remote group pushing ready for integration**  
Usman Akinyemi's feature enabling `git push` to handle remote groups (mirroring existing `git fetch` functionality) has addressed all review feedback and been queued by Junio. The implementation carefully maintains identical behavior to sequential individual pushes while adding comprehensive test coverage. The only noted limitation - lack of parallel push support - remains a potential future enhancement.

**In brief**  
**xdiff refactoring** -- Ezekiel Newren sends a narrowed-down v2 series improving readability in `xdl_cleanup_records()`, avoiding the more controversial structural changes from v1.

**Commit-graph date handling** -- Patrick Steinhardt's fix for generation number calculation with dates exceeding 34 bits is approved, with only minor platform-specific handling refinements remaining.

**Test modernization** -- Junio's 11-patch series to make the test suite work with `set -e` nears completion, with Patrick Steinhardt suggesting shell arithmetic improvements in t6002.

**Documentation polish** -- Christian Couder and Ben Knoble discuss improving `git replay` documentation to clarify root commit handling behavior and prevent accidental full-history replays.

**On the radar**  
**`add -p` discard feature** -- Luiz Campos' RFC for adding a discard-hunk action faces design questions about whether working tree modification belongs in `git add`, with Junio suggesting a more comprehensive interactive command may be preferable.

**Partial clone "un-fetch"** -- Siddharth Shrimali's GSoC proposal to allow moving blobs back to promised-but-not-present state shows promising technical understanding of Git's promisor machinery.

**Graph lane limiting** -- Pablo Sabater's `--graph-lane-limit` feature for `git log --graph` receives structural feedback from Johannes Sixt about patch organization and test philosophy.