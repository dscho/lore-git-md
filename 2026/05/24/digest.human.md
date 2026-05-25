# Here's the daily digest for May 24, 2026:

## The day in brief

A moderately busy Sunday with 29 emails across 15 threads, featuring performance optimizations, documentation refinements, and several patch series nearing completion. The standout items include a significant `git fetch` performance improvement and ongoing work on sparse-index optimizations.

## Notable threads

### Performance boost for `git fetch` connectivity checks

Kristofer Karlsson from Spotify identified and fixed a missed optimization in `git fetch`'s connectivity check that dramatically improves performance in large repositories. By passing the transport parameter through to `check_connected()`, Git can now skip verifying objects already in newly received packs. Benchmark results show connectivity check time dropping from 22s to 5s when fetching 200 new commits in a repository with 2.4M commits. Junio Hamano approved the change after confirming the optimization is well-established (dating back to 2013) and recently refined.

### Sparse-index optimization for `git restore`

Derrick Stolee continued his work on sparse-index optimizations with a two-patch series improving `git restore --staged` behavior. The changes avoid unnecessary index expansion when using `..` pathspecs by operating directly on tree entries rather than expanding to blob paths. Junio Hamano reviewed both the test coverage and implementation patches, suggesting a minor refactoring to improve code organization while approving the technical approach. This follows Stolee's characteristic test-first methodology for sparse-index changes.

### Commit-reach algorithm optimizations

Kristofer Karlsson (different from the fetch optimization author) proposed a three-patch series optimizing commit-reach.c algorithms used by `git merge-base` and ahead-behind calculations. The changes replace O(n) linear scans of commit queues with O(1) counter-based tracking, demonstrating 2.5x speedups in a 2.4M commit monorepo. Junio Hamano noted a potential flag collision concern but confirmed the immediate change is safe, while highlighting a broader need to update flag documentation in object.h.

### Shell completion behavior for dotfiles

Zakariyah Ali proposed changing Git's path completion behavior to hide dotfiles (like .gitignore) by default unless explicitly requested with a leading dot, aligning with standard shell conventions. Junio Hamano raised substantive concerns about whether this change is justified, particularly arguing that tracked dotfiles should complete like regular files. The discussion highlights Git's careful consideration of even small behavior changes that might affect user workflows.

## In brief

**Documentation refinements for git-interpret-trailers** -- Kristoffer Haugsbakk followed up on his recently merged 9-patch series with a polish-level change consolidating adjacent paragraphs about trailer appending behavior to improve clarity.

**git-gui repository detection improvements** -- Johannes Sixt reviewed the final patch in a series making git-gui's repository and worktree discovery more predictable, confirming the new `gui` and `pick` subcommands properly handle error cases.

**Line-log integration with standard diff pipeline** -- Michael Montalbo responded to review feedback on his series unifying `git log -L` with Git's standard diff output, agreeing to improve test hygiene and clarify commit messages while deferring stat format support.

**git commit --fixup message handling** -- Erik Cervin Edin addressed reviews about his patch allowing `-m` and `-F` options with `--fixup=amend:` and `--fixup=reword:`, agreeing to terminology changes and considering extending `-F` support to all `--fixup` variants.

**Documentation hook improvements** -- Junio C Hamano confirmed Kristoffer Haugsbakk's documentation patch series about Git hooks meets project standards and will be included in the next integration cycle.

**diff.<driver>.process protocol refinements** -- Michael Montalbo and Junio Hamano discussed implementation details for integrating external diff hunks, focusing on encapsulating xdiff's memory management details behind cleaner interfaces.

**git subtree split cache fix** -- A contributor proposed a fix for `git subtree split` failing on merge commits that make the same commit reachable via multiple paths, making the internal cache idempotent for identical mappings.

## On the radar

**Path-walk filter integration** -- Derrick Stolee's series adding filtering capabilities to Git's path-walk traversal is nearing readiness after Taylor Blau completed reviewing version 5 of the 14-commit series.