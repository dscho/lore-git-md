Here's the daily digest for September 4, 2025:

---

### The day in brief
A high-volume day with 101 emails across 21 threads, dominated by major architectural discussions around Rust adoption and commit graph refactoring. The Rust infrastructure proposal sparked extensive debate about build system requirements and platform compatibility, while Patrick Steinhardt's `git-history` command series reached RFC v3 with 18 patches. Junio Hamano's "What's cooking" report highlighted several new topics moving through the pipeline.

---

### Notable threads

**Rust adoption roadmap takes shape**  
Patrick Steinhardt kicked off a major RFC series to introduce Rust as a mandatory dependency for Git 3.0 (targeting late 2026), starting with build system support and a varint.c conversion as a test balloon. The proposal immediately sparked debate about meson version requirements (brian m. carlson noting current Debian ships meson 1.7.0 vs the proposed 1.9.0 minimum) and build system fragmentation (Makefile support being notably absent). Ezekiel Newren countered with a Cargo-based approach, while Eric Sunshine argued the optional implementation won't properly surface platform issues. Junio Hamano remained noncommittal but acknowledged platform policy concerns.

**Commit graph ownership challenged**  
Steinhardt's 6-part series to move commit graph handling from object_database to odb_source faced pushback from Junio, who argued commit graphs logically span the entire object store rather than individual sources. The technical refactoring was cleanly executed, but the maintainer questioned the fundamental design premise, suggesting graphs might need to cover history ranges that span source boundaries. This debate represents a pivotal moment for the pluggable ODB effort's architecture.

**git-history command matures**  
Steinhardt's RFC v3 of the Jujutsu-inspired history editing command now includes `drop`, `reorder`, `split`, and `reword` subcommands built on extensive sequencer infrastructure changes. The 18-patch series introduces tree-based status collection, in-memory index operations, and proper hook execution while maintaining compatibility with existing workflows. A diff visibility issue reported by D. Ben Knoble remains unresolved but doesn't block the overall design.

**Default branch change finalized**  
Phillip Wood's series to make "main" the default branch name (when built with WITH_BREAKING_CHANGES) received final approval after addressing reftable test dependencies. The 4-patch set updates documentation and test infrastructure while maintaining backward compatibility, with Junio noting the change will be properly announced in Git 2.52's release notes. This concludes a multi-year deprecation process started in 2020.

**Slab allocator API cleanup**  
A dangling pointer fix for the slab allocator wrapped up after resolving the final debate about NULL pointer handling philosophy. The v5 patch follows Git's convention of failing visibly on programming errors while making the API more robust through atomic cleanup operations. Jeff King and Junio converged on gracefully handling NULL values while strictly enforcing double-pointer contracts.

---

### In brief

**Upload-pack ACK deduplication fix** -- Patrick Steinhardt resolved Junio's concern about parent commit marking in protocol v2, restructuring `do_got_oid()` to ensure proper THEY_HAVE flag propagation while maintaining deduplication benefits.

**repo-info enhancements** -- Lucas Seiki Oshiro's series adding `-z` shorthand and object format reporting to `git repo info` got final approval after style fixes, now queued for 'next'.

**Partial clone workflows** -- Derrick Stolee noted `scalar clone` already implements blobless-by-default behavior, sparking discussion about feature graduation paths from Scalar to core Git.

**combine-diff recursive behavior** -- A bugfix for `git last-modified` exposed backward compatibility questions about `diff-tree -c` output formatting, with Junio requesting explicit `-r` documentation if behavior changes.

**macOS performance mystery** -- D. Ben Knoble's investigation into self-compiled Git slowdowns shifted focus to corporate security tooling interference after build optimizations failed to explain the gap.

**Type safety improvements** -- René Scharfe and Junio refined object ID abbreviation code to eliminate `void*` usage in `extend_abbrev_len()`, making the interface more compiler-checkable.

---

### On the radar

**Rust toolchain consensus** -- The build system debate continues with meson version requirements and Makefile support as key sticking points before Rust infrastructure can progress.

**Commit graph architecture** -- Junio's critique of the odb_source approach leaves open whether commit graph handling will need a redesign or can proceed with clarified semantics.

**Partial clone defaults** -- The discussion around making blobless clones more discoverable may lead to new `clone` options or documentation highlighting `scalar clone` as the current optimized path.