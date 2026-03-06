Here's the daily digest for September 22, 2025:

## The day in brief

September 22 saw significant activity across multiple fronts in Git development, with 99 emails across 21 threads. The day was dominated by ongoing discussions about Rust integration, platform compatibility concerns, and several bugfix series nearing completion. Key highlights include resolution on Rust infrastructure sequencing, progress on xdiff refactoring for Rust compatibility, and important fixes for stash behavior and memory management.

## Notable threads

**Rust integration coordination reaches consensus**  
Patrick Steinhardt and Ezekiel Newren finalized coordination on their parallel Rust integration efforts, agreeing that Steinhardt's minimal foundational patches (varint implementation and BreakingChanges documentation) will form the base for Newren's more comprehensive follow-up work. They resolved remaining questions about crate naming (`gitcore` was chosen) and type naming conventions, though Junio Hamano raised ongoing concerns about the aggressive timeline for making Rust mandatory in Git 3.0, particularly for platforms lacking Rust support.

**Dangling symref deletion edge case sparks API design discussion**  
A follow-up to Jeff King's dangling symref fixes revealed an edge case where Gitaly workflows intentionally delete dangling symrefs via null-to-null updates. The thread evolved into a broader discussion about `update-ref` API design, questioning whether `symref-delete` should remain a separate command or if `update` should handle more symref operations. While the technical fix is straightforward, the conversation highlighted deeper questions about refs API semantics that may need future attention.

**Xdiff refactoring for Rust compatibility advances**  
Ezekiel Newren's xdiff internals cleanup series reached v4 with 12 patches simplifying data structures and memory management in preparation for Rust integration. The changes remove redundant fields, convert character flags to proper boolean types, and eliminate the `chastore_t` allocator, resulting in neutral net performance impact. Junio Hamano provided final feedback about type safety in the boolean conversion, suggesting keeping the array as `char` to properly handle the three-state MAYBE value used in some diff operations.

**Stash index configuration finalized**  
A series adding configurable index restoration to `git stash` (via `stash.index`) completed its review cycle with all maintainer feedback addressed. The feature allows `git stash apply/pop` to restore the index by default when configured, while maintaining `--no-index` as an opt-out. Phillip Wood confirmed the changes look good, noting the config will implicitly affect `--autostash` behavior in merge/pull/rebase operations.

**In brief**

**Push optimization discussion** -- Sainan and Jeff King analyzed cases where Git unnecessarily resends blobs during push operations, particularly with amended commits. Potential solutions include smarter heuristics or leveraging reachability bitmaps.

**Gitk window focus fix** -- Johannes Sixt picked up a long-standing patch to make gitk's macOS window focusing more robust by gracefully handling missing `osascript` commands.

**Config color handling fixes** -- A series fixing `git config get --type=color` behavior completed, addressing empty key handling, error reporting, and pager spawning issues.

**Format-patch notes consistency** -- A bugfix series addresses inconsistent Git notes display between commit messages and range-diff output in single-commit cases.

**Case-insensitive lockfile warning** -- A documentation patch proposes improved error messages for lockfile collisions on case-insensitive filesystems.

**On the radar**

**SHA-1/SHA-256 interoperability** -- Brian m. carlson continues work on hash algorithm conversion, with recent focus on tag signature validation rules.

**Scalar config discoverability** -- Patrick Steinhardt and Derrick Stolee are coordinating on marking Scalar-set configs while documenting rationale externally.

**Deprecation warnings** -- Users seek clarification about `git whatchanged` and date-filtered `git log` deprecations, with alternatives provided for `whatchanged`.