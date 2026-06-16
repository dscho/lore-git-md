# Here's the daily digest for June 15, 2026:

**The day in brief.** A moderately active day with 111 emails across 28 threads, featuring several notable developments. Key highlights include the completion of Patrick Steinhardt's `git history drop` series and Johannes Schindelin's Windows large object handling patches, along with ongoing discussions about rebase improvements and contributor documentation.

## Notable threads

### `git log --graph` visualization improvements

Pablo Sabater's series to handle commits with excluded parents reached a new milestone as Junio C Hamano engaged with the test methodology. While acknowledging the core functionality appears sound, Junio raised concerns about test brittleness when commit-graph affects traversal order. The discussion has shifted from implementation details to ensuring robust testing that verifies relationship representation rather than exact output matching.

### `git branch --delete-merged` reaches v15

Harald Nordgren's long-running series for safe branch cleanup completed its 15th iteration, now renamed from `--prune-merged` to better reflect its behavior. The final version includes all planned safety mechanisms, per-branch opt-out via config, and the crucial `--dry-run` preview capability. Phillip Wood provided extensive documentation and implementation feedback throughout the series' evolution, with all concerns now addressed.

### Windows large object handling approved

Johannes Schindelin's 7-part series to properly handle objects >4GB on Windows systems (where `size_t` differs from `unsigned long`) received maintainer acknowledgment from Junio Hamano. The changes, which have been production-proven in Git for Windows, systematically convert size handling across delta, packfile, and ODB subsystems. Patrick Steinhardt's thorough review ensured all technical concerns were resolved before Junio indicated he would queue the series.

### `git history drop` completes

Patrick Steinhardt's 10-patch series introducing a new `drop` subcommand to the experimental `git history` command reached its final form. The implementation allows removing a commit while replaying descendants on top of its parent, with extensive safety mechanisms and 537 lines of test coverage. The series also modernized Git's reset machinery as part of the effort, receiving positive feedback from Phillip Wood and Junio Hamano.

### Documenting contributor norms

Weijie Yuan's RFC series to improve documentation about patch review etiquette progressed with Patrick Steinhardt's feedback. The discussion models the very practices being documented, showing proper attribution and substantive discussion before rerolling. The thread has clarified that changes should focus on `SubmittingPatches` rather than `MyFirstContribution.adoc` due to redundancy.

## In brief

**Compiler macro modernization** -- Dominik Loidolt's series to modernize `compat/posix.h` compiler check macros received final acks from both Patrick Steinhardt and Junio Hamano, completing this effort to improve Clang's UNUSED attribute handling.

**Ref backend refactoring** -- Patrick Steinhardt's v2 series modernizing reference backend infrastructure was reduced to 8 patches after performance concerns, focusing on initialization improvements and memory leak fixes while deferring absolute path transitions.

**Commit-graph timestamp fix** -- Elijah Newren's fix for generation number calculation with future-dated commits was acked by both Patrick Steinhardt and Derrick Stolee, with discussion shifting to broader testing of future date handling.

**`git cat-file` performance** -- René Scharfe and Jeff King explored optimizations for batch formatting, with benchmarks showing 3-7% improvements from avoiding `strbuf_addf()` in common cases.

**Path handling standardization** -- A GSoC participant's series to standardize path formatting in repository information reached v4 with all technical feedback addressed, now focusing on test helper simplifications per Justin Tobler's review.

## On the radar

**Rebase `--squash` proposal** -- Harald Nordgren's proposed `--squash` option for `git rebase` faces questions about whether the functionality might better fit in the experimental `git history` command, per D. Ben Knoble's architectural suggestion.