# Git Mailing List Digest — 2025/01/27

**The day in brief.** A busy Monday with 88 emails across 19 threads, dominated by two major technical efforts: Christian Couder's promisor-remote/LOP series reaching v4 with protocol capability refinements, and Patrick Steinhardt's 19-patch series decoupling the reftable library from Git's core infrastructure. Notable progress also on Derrick Stolee's name-hashing optimization and several platform-specific fixes.

## Notable threads

### Promisor-remote protocol v4 ready

Christian Couder's Large Object Promisor (LOP) series reached v4 with significant refinements to the protocol v2 capability negotiation. The updated design introduces server-side `promisor.advertise` and client-side `promisor.acceptFromServer` configurations, allowing servers to suggest alternate object sources while maintaining security boundaries. Junio Hamano raised final concerns about test isolation with MIDX interactions and patch management strategy for shared foundation code with Usman Akinyemi's parallel work. The series appears technically sound but awaits resolution on these process considerations before merging.

### Reftable library independence

Patrick Steinhardt submitted a comprehensive 19-patch series to fully decouple the reftable library from Git's core infrastructure. The changes systematically replace Git-specific helpers (like `read_in_full`, `write_in_full`, and `BUG()`) with reftable-specific implementations or direct POSIX calls. While the technical work is thorough, Junio raised important questions about long-term maintenance of these parallel implementations, suggesting the project may need a shared library approach to avoid divergence. The series represents a major architectural shift for reftable's future as a standalone component.

### Name-hashing optimization finalized

Derrick Stolee's performance optimization series introducing configurable name hashing for pack-objects delta compression reached its final form. The v4 implementation offers `--name-hash-version` with two algorithms: version 1 preserves existing behavior while version 2 improves directory hierarchy awareness. Performance tests show dramatic improvements in pathological cases (37GB → 7GB in one internal repo) while maintaining compatibility. Junio approved the series with only a minor documentation typo fix noted.

### Windows reftable compatibility

Ongoing discussion about Windows file handling in the reftable backend saw Patrick Steinhardt defending his approach to immediately fail on file-in-use errors rather than implementing retry logic. The debate centers on whether to modify Git's platform layer or adapt reftable code, with Patrick arguing the library can tolerate failed deletions gracefully. Independent testing confirmed the proposed solution works, though architectural placement questions remain open.

### Bare repository fetch fix approved

Bence Ferdinandy's fix for bare repository fetch behavior regression received final approval from both Patrick Steinhardt and Junio Hamano. The solution properly distinguishes mirror vs non-mirror remote behavior when handling HEAD references, preventing incorrect overwrites in bare repos. The patch series incorporated architectural improvements from review feedback and will be queued for integration.

## In brief

Christian Couder responded to Junio's suggestion about merging OS version capability with the "agent" string, arguing for separation to avoid browser-style User-Agent pitfalls. The reftable bugfix for uninitialized `max_index` during multi-worktree reference updates was queued after procedural updates. Meet Soni's refspec code reorganization series moved query and application functions to refspec.c with improved documentation, receiving detailed API quality feedback from Junio. Sören Krecker's MSVC warning fixes prompted discussion about proper type usage for line numbers, with Junio questioning whether converting to `size_t` is architecturally sound. The zlib-ng optimization series saw minor refinements to constness compatibility and stub implementations.

## On the radar

The promisor-remote/LOP series' shared foundation patches may require coordination with Usman Akinyemi's work before final integration. Junio's question about long-term maintenance of reftable's parallel implementations remains open for broader discussion among libification contributors. The Windows reftable file handling debate continues to converge on an architectural approach balancing platform specifics with library independence.