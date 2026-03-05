# Git Mailing List Digest - 2025/09/05

**The day in brief.** A busy day with 106 emails across 20 threads, dominated by ongoing discussions about Rust infrastructure adoption and several bugfix series nearing completion. Key developments include Patrick Steinhardt's v2 RFC for Rust integration, resolution of the slab allocator API cleanup, and continued debate about Git's version string format. The Rust transition timeline remains the most contentious topic, while multiple technical fixes show consensus forming.

## Notable threads

### Rust infrastructure RFC v2

Patrick Steinhardt's RFC series for Rust infrastructure saw significant discussion today, with v2 introducing Makefile support and a formal transition plan in BreakingChanges.adoc. The proposal outlines a 4-phase adoption:

1. Git 2.52: Optional auto-detection (Meson) / disabled (Makefile)
2. Git 2.53: Enabled by default for breaking changes builds
3. Git 2.54: Enabled by default for all builds
4. Git 3.0: Mandatory requirement

Technical refinements include relaxed meson version requirements (0.42.0+), a `WITH_RUST` preprocessor define, and CI testing expansion. The varint.c conversion demonstrates Rust/C interop with performance parity confirmed by Junio Hamano's assembly analysis.

Policy discussions focused on LTS support for non-Rust platforms, with Phillip Wood and Eli Schwartz advocating for a time-bound 2-3 year window rather than tying support to gcc-rs readiness. Build system questions narrowed to specific meson implementation details, while the fundamental approach gained technical validation.

### Slab allocator API finalized

After six iterations, the slab allocator cleanup series reached consensus with v6 approved for merging. The patch introduces `alloc_state_free_and_null()` to combine clearing and freeing operations atomically, fixing potential undefined behavior from cleared-but-not-freed states. Careful discussion resolved NULL pointer semantics, ensuring the API enforces contracts while safely handling idempotent calls.

Junio Hamano and Jeff King confirmed the final design properly distinguishes programming errors (NULL double pointer) from valid NULL inputs through immediate dereferencing and local variable checks. The changes improve safety in object.c callers while maintaining C90 compliance.

### MIDX write fixes ready

Derrick Stolee's multi-pack-index bugfix series addressing segfaults during repack/expire operations is now queued for 'next' after addressing Patrick Steinhardt's review feedback. The v3 patches:

1. Fix uninitialized packfile access during MIDX operations
2. Restore a dropped error code
3. Standardize error handling paths
4. Convert `preferred_pack_idx` to uint32_t
5. Remove signed comparison warnings
6. Simplify error case management

The changes resolve crashes reported by users running background maintenance with many (100+) packfiles, particularly visible under SANITIZE=address. Test modernization using `test_grep` and improved type safety were key review-driven improvements.

## In brief

**Sparse-checkout refactoring** -- Junio Hamano decoupled Derrick Stolee's dependent work from Ayush Chandekar's ongoing sparse-checkout variable refactoring, allowing Stolee's changes to progress while Chandekar's series awaits rebase.

**Protocol v2 ACK deduplication** -- Patrick Steinhardt's fix for redundant ACKs in upload-pack protocol v2 was approved, preventing memory growth from duplicate object IDs while maintaining backward compatibility with protocol v0.

**Subtree merge fix** -- A regression in squashed subtree merges under prefixes was fixed for maint-2.44, replacing `git log` calls with `git show` for proper trailer parsing during split operations.

**HP-UX test fixes** -- Michael Osipov addressed mkdtemp() linking failures in unit tests on HP-UX, though discussion continues about whether to modify Git's clar test framework copy or seek upstream changes.

**Version string debate** -- Jonas Rebmann's proposal to change snapshot version formatting from dotted (2.51.0.178) to standard git describe output (2.51.0-178) met resistance from Junio Hamano and brian m. carlson citing Git's non-SemVer tradition and 17-year precedent.

**`git last-modified` performance** -- Toon Claes fixed a BUG() in criss-cross merge handling while improving performance 2x by correcting recursive flag handling in diff_tree_combined().

## On the radar

**Blobless clone defaults** -- Дилян Палаузов's proposal to make blobless clones the default saw pushback from Konstantin Ryabitsev citing potential server load impacts comparable to shallow clones.

**Deprecated command usage** -- Vianney reported still actively using `git whatchanged`, reinforcing that deprecation may need clearer migration guidance despite functional equivalents existing.