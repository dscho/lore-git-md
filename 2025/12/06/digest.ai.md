# Git Mailing List Digest - December 6, 2025

**The day in brief.** A busy Friday with 49 emails across 11 threads, dominated by two major technical efforts: Taylor Blau's 17-patch MIDX compaction series and René Scharfe's security-focused tempfile hardening. Notable discussions include final refinements to the submodule path encoding design and a promising performance optimization for promisor pack handling.

## Notable threads

### MIDX compaction series lands

Taylor Blau introduced a comprehensive 17-patch series implementing **MIDX layer compaction**, a key component of Git's incremental repacking strategy for large repositories. The series enables combining adjacent MIDX layers while preserving bitmap validity, avoiding expensive full rewrites. The implementation maintains geometric progression of object counts across layers while keeping chain lengths bounded (typically under 10 layers).

Key technical aspects include:
- Relaxing lexicographic pack ordering requirements
- Introducing helper functions for pack permutation handling
- Adding bitmap support during compaction
- Extensive test coverage in t5335

The series follows a careful refactoring approach, with early patches improving const-correctness, documentation, and test infrastructure before introducing the core compaction logic. Performance benefits are significant for repositories with long MIDX chains, though the cover letter notes opportunities for additional test coverage.

### Secure tempfile infrastructure finalized

René Scharfe's security-focused series to eliminate insecure `mktemp(3)` usage reached completion with v2 patches that:
1. Introduce `git_mkdtemp()` as a secure alternative
2. Migrate Windows compatibility layer
3. Remove obsolete `mingw_mktemp()`
4. Ban `mktemp(3)` via `banned.h`
5. Eliminate redundant `gitmkdtemp()` wrapper

The changes represent a net reduction of 33 lines while improving security by removing race conditions in temporary file/directory creation. The thread shows careful consideration of build system impacts and third-party code dependencies, particularly around CMake on macOS. While focused on `mktemp`, the discussion leaves open whether to similarly ban `mkdtemp(3)` system calls in future work.

### Submodule path encoding design consensus

The long-running submodule gitdir path encoding discussion reached architectural consensus on several key points:
- Configuration will be purely runtime via `/etc/gitconfig`, rejecting build-time options
- An atomic enablement command will handle migration by persisting gitdirs to config
- Strict config authority will be maintained, erroring on missing entries rather than falling back

Patrick Steinhardt and Junio Hamano worked through edge cases around post-initialization enablement, settling on an approach that prevents loopholes while providing clear migration paths. The thread shows the project's careful balancing of strict design principles with practical adoption concerns.

### Promisor pack optimization breakthrough

Aaron Plattner's optimization for promisor pack handling demonstrated dramatic improvements, reducing processing time for a 176GB pack from 76 minutes to under 2 minutes. The key insight was skipping unnecessary decompression/hashing for blobs, which don't reference other objects. Discussion refined the approach to properly handle `OBJ_NONE` objects from `mark_uninteresting()` and consider safety tradeoffs around hash verification.

## In brief

**Windows symlink test fixes** -- Johannes Schindelin and Junio Hamano finalized test suite adjustments for Windows symlink handling, clarifying that behavior should align with POSIX standards rather than Linux specifically.

**Object database leak fix** -- Patrick Steinhardt addressed a 192-byte memory leak in submodule handling during git-grep operations, with discussion focusing on commit message clarity.

**Clar test framework updates** -- Patrick modernized Git's unit test infrastructure with type-safe integer comparisons and new relative assertion macros, plus whitespace handling fixes for test output files.

**Structured data version control** -- The thread evaluating Git modifications for structured data versioning expanded to consider DVC, Dolt, and Fossil SCM as potential alternatives, with Cedric Sodhi analyzing their architectural tradeoffs.

## On the radar

**Histogram diff bugfix** -- A new patch addresses rare but confusing redundant output in histogram diffs when shifted change groups include matching lines, with thorough test coverage demonstrating the fix. This subtle but impactful change merits watching as it moves through review.