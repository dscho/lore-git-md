# Git Mailing List Digest - 2026/04/10

## The day in brief

A busy day with 96 emails across 14 threads, dominated by major feature finalizations and platform compatibility discussions. The parallel hook execution series reached completion with maintainer approval, while build system improvements and Rust integration debates continued. Notable highlights include the merge-ready parallel hooks feature and ongoing discussions about `writev()` optimizations and Git 3.0 version numbering.

## Notable threads

### Parallel hook execution finalized

Adrian Ratiu's parallel hook execution series (v7) received final approval after incorporating Jeff King's SIGPIPE test. This major feature enables concurrent hook execution with extensive configuration options:
- Global (`hook.jobs`), per-hook (`hook.<name>.parallel`), and CLI (`-j/--jobs`) controls
- Special handling for pre-push hooks with automatic output merging
- Seven commit/checkout-related hooks remain permanently serial for safety
- Comprehensive test coverage including SIGPIPE handling validation

The implementation builds on Ratiu's earlier configurable hooks work and has been thoroughly reviewed by both Junio Hamano and Patrick Steinhardt. This represents a significant performance improvement for hook-heavy workflows while maintaining backward compatibility.

### Build system improvements near completion

Patrick Steinhardt's build system series addressing tools/ directory reorganization and meson PCH support is in final refinement. The last technical discussion focuses on precompiled header (PCH) implementation details:
- Confirmed GCC documentation specifies PCH inclusion limitations
- Concern raised about implicit inclusion in reftable sources
- Phillip Wood supports respecting architectural boundaries
- Series appears ready for merge pending this final adjustment

The changes promise 30-40% build speed improvements while maintaining cross-platform compatibility. The thorough review process demonstrates Git's attention to both performance and maintainability concerns.

### Git 3.0 version numbering debate

The Rust integration thread expanded into a broader discussion about Git's release numbering strategy. Key points:
- brian m. carlson assumed Git 3.0 would follow 2.55 (targeting Sept 2026)
- Junio Hamano expects to reach 2.95 first for clearer signaling
- Derrick Stolee suggested delaying Rust-by-default to 2.55
- Junio proposed process changes for high-impact features

The technical implementation of Rust support remains uncontroversial, but the version numbering and release timing discussion reveals differing expectations about Git's development roadmap.

### NonStop platform compatibility issues

Multiple threads addressed NonStop platform constraints:
- `writev()` optimization conflicts with NonStop's small `MAX_IO_SIZE`
- Patrick Steinhardt's `xwritev()` solution deferred post-2.54
- Johannes Sixt questioned `writev()` optimization value entirely
- Full revert of `writev` changes planned for 2.54 release

These discussions highlight the tension between performance optimizations and platform compatibility in Git's cross-platform ecosystem.

## In brief

**Promisor file repack handling** -- Lorenzo Pegorari's v5 series preserves debugging info in `.promisor` files during repacks, now with memory leak fixes and proper timestamp handling.

**Autostash for checkout -m** -- Harald Nordgren's v9 series brings autostash behavior to branch switching, with renamed label options and comprehensive test coverage.

**In-memory ODB backend** -- Patrick Steinhardt's series implementing transient object storage concludes with terminology clarification ("in-memory" vs "in-core").

**Bundle-URI protocol fixes** -- Discussion converges on combining client-side validation with server-side filtering for invalid bundle-URI data.

**GSoC mentor recruitment** -- Call for additional mentors to handle 19 student proposals, emphasizing how mentoring grows Git's contributor base.

## On the radar

**Rust integration timeline** -- Build system changes to enable Rust by default now targeted for Git 2.55, pending version numbering consensus.

**NonStop platform support** -- `writev()` optimizations remain deferred post-2.54 while architectural solutions are developed.

**Git 3.0 planning** -- Version numbering strategy needs clarification separate from Rust integration discussion.