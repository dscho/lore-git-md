# Git Mailing List Digest - 2025/09/05

**The day in brief.** A busy day with 106 emails across 20 threads, dominated by ongoing discussions about Rust infrastructure adoption and several bugfix series nearing completion. Key highlights include Patrick Steinhardt's v2 RFC for Rust integration, resolution of the slab allocator API cleanup, and continued debate about Git's version string format. The Rust transition timeline remains the most contentious topic, with multiple contributors weighing in on platform support and LTS strategies.

## Notable threads

### Rust infrastructure RFC v2

Patrick Steinhardt's RFC series for introducing Rust infrastructure to Git core reached version 2, now including Makefile support alongside meson implementation. The series demonstrates Rust integration by converting the varint subsystem while maintaining C ABI compatibility. Key additions include:

- `WITH_RUST` preprocessor define for build status reporting
- Relaxed meson version requirement from 1.9.0 to 0.42.0
- Documentation of a 4-phase transition plan in BreakingChanges.adoc
- Expanded CI testing with full build-and-test runs

The discussion focused on refining the transition timeline and LTS strategy, with Phillip Wood advocating for a 6-month mandatory adoption period rather than tying it to Git 3.0. Platform support concerns (particularly for NonStop) remain unresolved, though consensus is forming around a time-bound 2-year LTS window for non-Rust platforms.

### Slab allocator API cleanup finalized

After multiple iterations and thorough review, the slab allocator API improvements were approved and queued for merging. The series:

- Renames functions to follow Git conventions (`alloc_state_alloc`)
- Combines clearing and freeing into `alloc_state_free_and_null()`
- Adds proper NULL pointer handling while maintaining API contract enforcement
- Updates all callers in object.c

The thread demonstrated Git's careful API design process, with extensive discussion about NULL pointer semantics and C90 compliance. Jeff King and Junio Hamano both signed off on the final implementation, which balances safety with clear error detection.

### MIDX write bugfix series

Derrick Stolee's series addressing a segfault in multi-pack-index operations reached version 3 with all feedback incorporated. The fixes:

- Correct uninitialized memory access during packfile closing
- Standardize error handling patterns in midx-write.c
- Improve type safety by converting `preferred_pack_idx` to `uint32_t`
- Modernize tests to use `test_grep` instead of raw `grep`

The changes address a concrete issue affecting users since Git 2.47.0, particularly visible during background maintenance with many packfiles. The robust test case (requiring 100 packfiles to reproduce under SANITIZE=address) provides confidence in the solution.

## In brief

**Protocol v2 ACK deduplication** -- Patrick Steinhardt's patch series fixes redundant ACKs in upload-pack when clients send duplicate "have" lines, preventing unnecessary memory growth (~4MB/s savings).

**Sparse-checkout refactoring workflow** -- Junio Hamano decoupled Derrick Stolee's dependent work from Ayush Chandekar's sparse-checkout variable refactoring to unblock progress, with plans to rebase later.

**Subtree merge regression fix** -- Final v2 patch fixes squashed subtree merges under prefixes during split operations, addressing a regression introduced in commit 98ba49ccc2.

**HP-UX test build fixes** -- Michael Osipov addressed `mkdtemp()` linking failures in the unit test framework on HP-UX systems through proper header inclusion.

**Default branch name documentation** -- Phillip Wood finalized documentation tweaks for the merged series changing Git's default branch name to "main" when built with breaking changes.

**On the radar**

**Rust transition timeline** -- The policy debate continues around whether to tie mandatory Rust adoption to Git 3.0 or follow a faster 6-month timeline, with LTS support duration for non-Rust platforms still under discussion.

**Version string format** -- The debate over maintaining Git's historical dotted format versus switching to standard `git describe` output remains unresolved, with Junio Hamano favoring backward compatibility.