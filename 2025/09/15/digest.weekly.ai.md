# Git Mailing List Weekly Digest  
**2025/09/15 -- 2025/09/21**  

## The week in brief  

A highly active week with 614 emails across 152 threads saw major infrastructure work converging alongside heated debates about Git's future direction. The Rust integration effort reached a critical juncture with maintainer approval but unresolved platform compatibility concerns, while foundational changes to packfile storage and reference handling reached completion. Key developments include Patrick Steinhardt's packfile store refactoring landing, Justin Tobler's ODB transaction interface finalization, and definitive confirmation as NonStop platform constraints that may force Rust to remain optional.  

## Key developments  

### Rust integration approved but constrained  

The Rust infrastructure series received maintainer ack from Junio Hamano in its v5 iteration, marking a major milestone in Git's transition toward mandatory Rust support by version 3.0. However, definitive technical constraints emerged from NonStop platform maintainer Randall Becker, who confirmed Rust cannot be supported due to proprietary licensing and security audit requirements. Ezekiel Newren's efforts to minimize adoption barriers (targeting Rust 1.63.0, avoiding Cargo) proved insufficient for this platform, forcing the project to reconsider whether Rust can be mandatory. Meanwhile, build system challenges around PIC/PIE handling and Windows/MSVC compatibility remain active discussion points.  

### Packfile store refactoring completes  

Patrick Steinhardt's 15-part series centralizing packfile management in a new `struct packfile_store` reached completion after extensive review. This foundational work for pluggable ODB backends relocates all packfile-related state from `struct object_database` to the new store structure, touching core packfile operations across many commands. The final version addressed MIDX handling and transaction behavior concerns through careful API design. This architectural change enables future backend flexibility while maintaining performance characteristics of the existing implementation.  

### ODB transaction interfaces finalized  

Justin Tobler's 6-part series refactoring object database transaction handling established clean API boundaries in odb.[ch] while relocating implementation to object-file.[ch]. The changes simplify transaction nesting support and improve error handling, though some discussion continues about API simplification tradeoffs. This work complements Steinhardt's packfile changes by providing the transaction infrastructure needed for future pluggable backends, with particular attention to `update-index --verbose` edge cases during the transition.  

### Reftable validation matures  

Karthik Nayak's v3 series for reftable fsck validation introduced runtime checks for critical format requirements like sequential update indices and proper table naming. Junio Hamano guided the separation of checks into runtime validation versus advisory fsck warnings, resulting in a robust system that positions reftable for wider adoption. The implementation includes comprehensive test coverage and careful error classification, addressing a key stability requirement before reftable can become the default backend.  

### Ref optimization API established  

Meet Soni's backend-agnostic `refs_optimize()` API reached completion after multiple iterations, modernizing Git's ref storage operations. The implementation provides concrete files and reftable backend support while converting `git pack-refs` to use the new interface. A new `git refs optimize` subcommand offers identical functionality through a more intuitive name, with both commands sharing core logic via `pack_refs_core()`. This foundational work enables future backend-specific optimization strategies while maintaining backward compatibility.  

### Stash behavior improvements land  

D. Ben Knoble's series adding a `stash.index` config option was approved, making `stash apply/pop` restore the index by default when configured. This addresses a long-standing inconsistency where `stash push` saves staged changes but `apply/pop` doesn't restore them. The final version uses runtime configuration rather than compile-time breaking changes, resolving a common user pain point while maintaining backward compatibility. Meanwhile, Jeff King fixed a critical double-free` in `git stash show -p` through comprehensive strvec memory management improvements.  

## In brief  

**git-history command design** -- Patrick Steinhardt's RFC evolved with decisions to keep modified-files display (not full diffs) for initial implementation and plan message editing for both commits in a split. SZEDER Gábor raised concerns about automatically rewriting descendant commits in the proposed `drop` subcommand.  

**Worktree safety checks** -- Debate shifted from implementation to whether rebase should support `--ignore-other-worktrees` at all, with Junio Hamano advocating detached HEAD for experimental changes. Gabriel Scherer later proposed `--detach-other-worktrees` as a middle ground.  

**SHA-1/SHA-256 interoperability** -- brian m. carlson began documentation and test groundwork, confirming 32-byte SHA-256 checksums in pack indices and clarifying tag signature field handling.  

**Command deprecation framework** -- Kristoffer Haugsbakk's series implementing Git's command deprecation system completed with `git-whatchanged` exclusion from `BUILT_INS` when `WITH_BREAKING_CHANGES` is set.  

**Xdiff refactoring** -- Ezekiel Newren's preparatory cleanups merged, removing redundant fields and replacing the chastore_t allocator with a 5% faster contiguous array approach.  

**Color handling** -- Jeff King standardized Git's handling code, replacing numeric constants with proper enums and fixing grep/diff colorization bugs.  

**git add -p hunk splitting** -- Phillip Wood's behavior changes will use runtime configuration after Junio rejected version gating.  

**Shallow clone limitations** -- Elijah Newren clarified commit hash-based `--shallow-exclude` is "flawed" due to technical constraints, with blobless clones suggested as a workaround.  

**NonStop platform** -- Day-long discussion resolved test framework issues by adopting `%p` formatting for pointer comparisons after `uintptr_t` solutions proved architecture-dependent.  

**gitk improvements** -- Geometry persistence fixes for main and "Tags and Heads" windows, plus investigation of macOS focus issues when `osascript` is unavailable.  

## Looking ahead  

The Rust adoption timeline faces reconsideration after NonStop platform constraints made mandatory support impossible, with the project needing to decide whether to proceed with optional Rust or adjust version 3.0 plans. The semantic conflict resolution debate around `git-rebase-clean` remains paused pending Antonio Mennillo's response to Elijah Newren's fundamental workflow philosophy critique. Protocol-level fixes for `git fetch --negotiate-only` behavior with unrelated commits may see progress now that Kevin Puetz has identified the root state machine issues in upload-pack.c.