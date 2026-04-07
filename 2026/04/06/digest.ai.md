# Git Mailing List Digest - 2026/04/06

**The day in brief.** A moderately busy day with 70 emails across 23 threads, dominated by technical discussions around several significant features and bugfixes. Key developments include Junio Hamano's approval of the fsmonitor Linux implementation after resolving the split-index interaction issue, rejection of configurable detach behavior for `git switch`, and progress on multiple performance optimizations. The day also saw continued work on the ODB abstraction effort and several documentation improvements.

## Notable threads

### **Linux fsmonitor implementation finalized**

Junio Hamano provided the final approval for Paul Tarjan's v13 series implementing Linux filesystem monitoring via inotify, bringing it to parity with existing Windows and macOS support. The maintainer confirmed the fundamental incompatibility between split-index and index.skipHash features while endorsing the test-focused solution of unsetting GIT_TEST_SPLIT_INDEX in scalar clone tests. 

The series represents a significant milestone in cross-platform fsmonitor support, having addressed all substantive review feedback across 13 iterations. Key technical components include process isolation enhancements, a new daemon shutdown timeout, complete Windows pthread emulation, Linux inotify implementation with recursive watches, and memory leak fixes in both client and daemon paths. The implementation appears production-ready with thorough test coverage of edge cases.

### **Parallel hook execution ready for merge**

Adrian Ratiu's parallel hook execution series (v6) received final approval from Junio Hamano after addressing all review feedback. The implementation provides comprehensive parallelization controls through multiple configuration layers: global `hook.jobs` defaults, per-hook `hook.<name>.parallel` flags, CLI `-j/--jobs` overrides, and per-event `hook.<event>.jobs` settings. 

Seven critical hooks (commit-related and checkout-related) remain permanently marked non-parallelizable via `RUN_HOOKS_OPT_INIT_FORCE_SERIAL` to ensure safety. The series demonstrates careful attention to backward compatibility while significantly expanding hook system capabilities, with all core components now maintainer-approved.

### **Promisor file handling during repacks**

Lorenzo Pegorari's GSoC work on promisor file handling during repacks received detailed review feedback from Junio Hamano. While acknowledging the memory leak fix identified by Tian Yuchen, Junio raised broader concerns about using `strbuf_split_max()` in the implementation, suggesting a simpler `string-list` approach would better align with the project's string handling cleanup efforts.

The maintainer also clarified the timestamp rationale in `.promisor` files, noting modification times serve as useful proxies for when remote refs were fetched rather than representing creation times. The series remains in review while the author reworks the string parsing logic per maintainer feedback.

### **`git switch` detach behavior proposal rejected**

Junio Hamano firmly rejected Thibaud CANALE's proposal to make `git switch`'s detach behavior configurable via `checkout.switchDetach`. The maintainer argued this would undermine the command's design purpose as a simpler, branch-focused alternative to `git checkout`, stating "`git checkout` remains available for detach behavior."

Kristoffer Haugsbakk later added commentary noting personal preference for `git checkout` while acknowledging official guidance to use `git switch`/`git restore`, highlighting the tension between documentation labeling `checkout` as "superseded" and its continued use in practice.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Windows version requirements updated** -- Johannes Schindelin and Matthias Aßhauer's series finalizing Git for Windows' move to requiring Windows 8.1 as minimum version was accepted by Junio Hamano.

**`git history` help text fix** -- René Scharfe corrects inaccurate option documentation in the new `git history` command's help text.

**Cache-tree validation fix** -- David Lin addresses an inverted condition in `cache_tree_fully_valid()` that caused incorrect validation when tree objects were missing.

**Promisor-remote submodule fix** -- Trieu Huynh corrects repository context handling when reading `promisor.quiet` config during submodule operations.

## On the radar

**ODB write operations** -- Justin Tobler's series implementing the write operations interface for Git's object database transactions continues review, with Jeff King providing minor optimization suggestions on the streaming path implementation.

**Bare repository test modernization** -- Johannes Schindelin's effort to prepare tests for potential `safe.bareRepository=explicit` default saw Junio suggest an alternative approach using global `safe.bareRepository=all` configuration in test-lib.sh.

**`fast-export` output ordering** -- Discussion continues on Raymond E. Pasco's fix for path deletion ordering, with Elijah Newren providing key insights about `fast-import`'s last-entry-wins semantics strengthening the case for exporter-side ordering.