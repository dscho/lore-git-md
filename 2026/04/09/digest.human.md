# Git Mailing List Digest - 2026/04/09

**The day in brief.** A busy Thursday with 127 emails across 21 threads, dominated by platform compatibility discussions around NonStop's `writev()` limitations and significant progress on the ODB abstraction effort. Key highlights include Patrick Steinhardt's in-memory ODB source nearing completion and the fsmonitor Linux implementation receiving maintainer approval after 14 iterations.

##Notable threads

**fsmonitor Linux implementation approved**  
Paul Tarjan's 14-iteration series implementing Linux filesystem monitoring via inotify has received final approval from Junio Hamano. The comprehensive solution brings Linux to parity with existing Windows and macOS implementations, addressing memory leaks, process isolation, and test coverage. Key technical components include non-blocking event polling, recursive watch registration, and mount point detection via `/proc/mounts`. The series demonstrates thorough engineering with attention to edge cases like stale renames and watch limits, now marked "Will queue" for integration.

**ODB abstraction naming discussion**  
Patrick Steinhardt's 17-patch series introducing an in-memory object database source has progressed from technical implementation to a conceptual discussion about terminology. Junio Hamano questions whether "in-memory" aligns with Git's existing "in-core" terminology, while Patrick argues "in-memory" will be more intuitive for future user-facing configuration. The technical implementation has already been reviewed and confirmed by Justin Tobler, with this final naming discussion representing the last hurdle before integration. The series is a significant milestone in the long-running ODB abstraction effort.

**NonStop platform `writev()` limitations**  
A platform compatibility issue emerged when Git's `writev()` optimization failed on NonStop systems due to unusually small `MAX_IO_SIZE` limits (52KB for 32-bit). After extensive discussion, Junio Hamano reverted the entire `writev()` emulation layer as a temporary solution for Git 2.54, with plans to implement proper `MAX_IO_SIZE` support post-release. Jeff King proposed a more targeted revert that would preserve scaffolding for future reactivation, but Junio opted for a clean slate approach to ensure immediate stability. The thread highlights Git's pragmatic approach when platform constraints conflict with optimization attempts.

**Branch name prefixing design questions**  
Yoann Valeri's fourth iteration of a branch name prefixing feature faces escalating design objections from Junio Hamano. The maintainer questions both the fundamental value of the feature and specific usability concerns about hyphen handling in prefixed names. While the technical implementation has improved through review cycles, Junio's latest feedback suggests the series is unlikely to progress without either stronger justification or a redesign considering Eric Sunshine's earlier string interpolation suggestion. The exchange demonstrates how even well-implemented features can stall if they don't convince the project of their core value proposition.

**`git checkout -m` autostash finalized**  
Harald Nordgren's series enhancing `git checkout -m` with autostash behavior has reached its final approved version (v8) after addressing all remaining review feedback. The implementation now includes support for empty conflict labels, robust stash list verification, and comprehensive test coverage (19 cases in t7201-co.sh). Junio Hamano confirmed the series looks good after the addition of a new test case for non-overlapping changes. This represents a significant quality-of-life improvement for branch switching workflows, particularly when working with uncommitted changes.

##In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**xdiff refactoring integration** -- Phillip Wood verifies his `pw/xdiff-shrink-memory-consumption` topic rebases cleanly on Ezekiel Newren's xdiff cleanup series in 'seen'.

**Promisor-remote config fix** -- Trieu Huynh addresses incorrect repository context when reading `promisor.quiet` config during submodule operations, with test fragility under investigation.

**Rust enabled by default** -- brian m. carlson submits a 4-patch series to make Rust support the default in Git's build system, following through on plans originally slated for 2.53.

**`git stash` argument parsing** -- Deveshi Dwivedi's series to assume "push" subcommand when unambiguous flags are present reaches v4, with final discussion about hyphen handling ergonomics.

##On the radar

**Negotiation for critical branches** -- Derrick Stolee explores interface design alternatives for combining must-have negotiation with existing `--negotiation-tip` after Junio Hamano suggested unification.

**Signal handling architecture** -- The discussion following the SIGPIPE fix has uncovered systemic issues in Git's signal handler management, with Jeff King proposing architectural improvements as future work.

**`git repo info` path handling** -- Jayesh Daga's metadata feature faces design questions about path normalization behavior, with Karthik Nayak suggesting either configurable output or separate relative/absolute fields.