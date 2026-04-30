# Git Mailing List Digest - 2026/04/29 (Wednesday)

**The day in brief.** A moderately busy Wednesday with 35 emails across 15 threads, featuring several significant technical discussions and patch series reaching completion. The most notable developments include the finalization of the xdiff refactoring series, resolution of a `git fetch --deepen` regression, and continued work on Windows platform compatibility for large objects. Meanwhile, Taylor Blau's MIDX repacking series received its final polish before integration.

## Notable threads

### **xdiff refactoring reaches completion**

Ezekiel Newren's six-part series refactoring `xdl_cleanup_records()` in the xdiff subsystem has reached its final iteration (v6), incorporating all feedback from Phillip Wood. The changes focus on type safety and code clarity improvements, particularly around limit calculations and action execution logic. The series now uses separate variables (`mlim1`, `mlim2`) for each file's processing, replaces nested ternary operators with explicit if/else structures, and adds a BUG() check for illegal action states. With all review feedback addressed and no behavioral changes from the regression-fixed v5 version, this series appears ready for merging after its careful progression through multiple iterations.

### **`git fetch --deepen` regression identified**

Owen Stephens reported a concerning regression in Git 2.54.0 where repeated `fetch --deepen` operations on a fully-fetched shallow clone can incorrectly revert the repository to a shallow state, losing previously visible commits. The bug manifests specifically on ARM64 Darwin systems and was introduced in commit 3ef68ff. D. Ben Knoble noted documentation ambiguities around whether `--deepen` should be a no-op in non-shallow repositories, but the data loss aspect suggests fixing the regression takes priority over documentation clarification. The reporter provided thorough reproduction steps and version bisection, making this a high-quality bug submission that warrants prompt attention.

### **Windows large object handling advances**

Johannes Schindelin's series addressing Windows limitations with objects >4GB received review feedback from Derrick Stolee focusing on test execution time. The tests take approximately 4 minutes in CI, prompting discussion about whether to mark them as "expensive" or handle them differently between upstream Git and Git for Windows. The series itself systematically replaces 32-bit types with 64-bit `size_t` in critical paths (index-pack, unpack-objects, zlib wrapper, ODB streaming) to prevent truncation issues. While the technical approach has been approved, the test optimization discussion continues, balancing coverage needs with CI resource constraints.

### **MIDX repacking series finalized**

Taylor Blau's 16-patch series implementing incremental MIDX/bitmap-based repacking received its final polish from Jeff King (Peff), addressing minor documentation inconsistencies and process safety concerns. The changes ensure proper cleanup in error paths and correct flag name references in commit messages. With both Peff and Elijah Newren satisfied with the technical implementation, this major scalability improvement for large repositories now awaits maintainer integration. The series introduces two operational modes for MIDX chain management - geometric repacking (recommended) and append-only growth - with comprehensive test coverage.

## In brief

**`git subtree` cache fix** -- A bugfix addresses edge cases in the split cache mechanism when re-adding previously removed subtrees, preventing spurious "cache already exists" errors.

**Test socket path limitations** -- Johannes Schindelin's v2 patch fixes Unix domain socket path length issues in t5564-http-proxy.sh tests using secure `mktemp -d` directories.

**Git-gui bare repo handling** -- Shroom Moo simplified their fix for git-gui crashes in bare repositories, focusing on error handling around `rev-parse --show-toplevel` after reviewer feedback.

**Format-rev compiler warning** -- Kristoffer Haugsbakk addressed a CI-caught compiler warning about uninitialized variable use in the experimental `git format-rev` command.

**Maintenance error reporting** -- Anselm Schüler reported that `git maintenance` provides insufficient error details when remotes are unavailable, suggesting improvements to error messages and continuation behavior.

## On the radar

**Ref backend consolidation** -- Karthik Nayak's refactoring series needs to address Toon Claes' identified error handling issue in patch 6/9 before finalization.

**Tarball reproducibility** -- The ongoing discussion about `git archive` behavior continues to surface historical format changes that affect packaging workflows, with Jeff King providing additional implementation context.