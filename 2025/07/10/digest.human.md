Here's the daily digest for July 10, 2025:

## The day in brief

July 10 saw significant activity across multiple fronts, with 79 emails across 21 threads. The day was dominated by technical refinements to ongoing series - particularly Patrick Steinhardt's MIDX refactoring and the bloom filter optimizations - alongside discussions about community norms and platform-specific fixes. Key developments include the finalization of the daemon signal handling series and progress toward making reftable the default backend in Git 3.0.

## Notable threads

### MIDX tracking moves to per-source storage

Patrick Steinhardt's 8-part series to move MIDX tracking from global state to per-source storage saw extensive review from Taylor Blau and Justin Tobler. The changes restructure how Git handles multi-pack indexes to better align with the object database's source-based architecture. Reviewers validated the technical soundness while suggesting minor improvements to commit messages and code organization. The series completed its transition by removing all global MIDX infrastructure in the final patch, marking a significant step in the object storage refactoring effort.

### Bloom filter optimizations reach final polish

Lidong Yan's bloom filter optimization series for pathspec handling reached its final iteration (v5), with Derrick Stolee providing performance benchmarks showing 1.5-7.5x speedups. The changes enable bloom filter usage with multiple literal pathspecs while maintaining the fast path for single pathspec cases. Review focused on code organization, with Stolee suggesting extracting pathspec-to-bloom-key conversion into a helper function. Junio noted some now-unused variables from the refactoring, indicating the series is in its final cleanup phase before merging.

### Git daemon signal handling finalized

Carlo Marcelo Arenas Belón's signal handling series for `git daemon` was approved after addressing Windows compatibility concerns. The v4 patches transition from `signal()` to `sigaction()` while maintaining platform-specific behaviors, with Junio applying final typo fixes before queuing. The changes preserve BSD-style signal semantics and explicitly defer pipe-based process tracking to future work, marking the conclusion of this phase of daemon improvements.

### Community norms discussion continues

The thread about signature handling evolved into a broader discussion about community response norms, with Patrick Steinhardt, Christian Couder, and Junio Hamano exchanging views. Steinhardt advocated for employed contributors setting response time examples, while Couder cautioned against rigid expectations that could harm volunteer participation. Junio aligned with Steinhardt's position in a brief response, though the discussion remained constructive without clear resolution.

## In brief

**Reflog memory leak fix** -- Jacob Keller and Jeff King refined a fix for memory leaks in reflog expiration, addressing cleanup timing issues during multi-ref operations.

**Meson build improvements** -- Patrick Steinhardt's Meson series saw positive reviews, particularly for colorized feature output, while Ramsay Jones proposed an alternative fix for libexecdir handling.

**HTTP language tag compliance** -- Brian Carlson proposed filtering invalid "C" and "POSIX" locale values from Accept-Language headers for standards compliance.

**VS Code scissor-line bug** -- Investigation continued into unexpected `commit --verbose` behavior with VS Code, with testers unable to reproduce using default settings.

**Amazon Linux 2 build fix** -- Patrick Steinhardt addressed header inclusion conflicts causing build failures on Amazon Linux 2 with GCC 7.3.1.

**Release versioning proposal** -- Junio Hamano RFC'd changing to time-based monthly tags to encourage distros to update more frequently, though brian m. carlson expressed skepticism about effectiveness.

## On the radar

**SHA-256 default** -- brian m. carlson's series to make SHA-256 the default in Git 3.0's breaking changes mode is queued for 'next' after positive reviews.

**Pluggable ODB backends** -- Patrick Steinhardt's series to make the loose object backend pluggable continues with positive reviews, though a regression in fast-import was noted.

**Reftable default** -- Junio's "What's cooking" notes reftable will become the default format for new repositories in Git 3.0 when experimental features are enabled.