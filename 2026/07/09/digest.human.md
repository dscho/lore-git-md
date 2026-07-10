# The Git Mailing List Daily Digest for 2026/07/09

## The day in brief

A busy Thursday with 108 emails across 23 threads. The standout items: a **performance regression fix** for the reftable backend (now in v3), **ODB abstraction progress** with object filters for `odb_for_each_object()`, and **new worktree hooks** for tooling integration. Architectural debates continue on **external notes commands** and **`includeIf worktree:`**, while routine refactoring and bugfixes advance the `the_repository` removal effort.

---

## Notable threads

### Reftable backend performance regression fix (v3)
**Kristofer Karlsson** posted the third iteration of a two-patch series fixing a quadratic performance regression in the reftable backend when tombstones (deleted refs) are present. The regression occurs because the merged iterator’s `suppress_deletions` flag forces a full scan of all tombstones before bounds checks can terminate iteration. The fix moves tombstone handling to call sites where prefix and refname bounds are available, enabling early termination. Performance improves from ~14s to ~0.2s in the perf test’s 8000-ref scenarios.

Patrick Steinhardt (reftable maintainer) and the author agreed on the final API shape: the `suppress_deletions` flag moves to `struct reftable_stack_options`, preserving libgit2’s current behavior while future-proofing the API. The series is now uncontested and ready for `next`.

---

### ODB abstraction: object filters for `odb_for_each_object()`
**Patrick Steinhardt** posted a seven-patch series introducing object filters to `odb_for_each_object()`, enabling `git cat-file --batch-all-objects` to work with pluggable ODBs. The series replaces backend-specific logic in `cat-file` with ODB-mediated filtering, removing 69 lines of code and eliminating redundant index traversals during enumeration.

Junio C Hamano praised the final patch as "very nice," while Justin Tobler raised design questions about the `filter` field in `odb_for_each_object_options`—whether backends should be required to support a subset of filters or if callers should re-apply unsupported filters. The series is likely to land once the commit message for the final patch is tweaked to better motivate the change.

---

### New worktree hooks: `post-worktree-add` and `post-worktree-remove`
**Domen Kožar** introduced a three-patch series adding two new hooks to Git’s worktree subsystem. The hooks address a gap in automation tooling (particularly AI-driven development environments) that create and destroy worktrees at high velocity. `post-worktree-add` runs after `git worktree add` completes, regardless of checkout status, while `post-worktree-remove` runs after `git worktree remove` or pruning. Both hooks receive the worktree’s absolute path and identifier as arguments.

The series is well-motivated and includes comprehensive test coverage. Two design questions are flagged for discussion: whether `post-worktree-add` should run if `post-checkout` fails, and how to handle path resolution for pruned worktrees when the gitdir file is missing. The hooks align with Git’s existing hook semantics and should be uncontroversial.

---

### External notes commands: architectural limbo
**Siddh Raman Pant**’s four-patch series adding support for external commands to display Git notes content remains stalled in an architectural debate with Johannes Sixt (Hannes). The series enables Git to delegate notes content retrieval to external commands via a new protocol, addressing TOCTOU issues in large repositories. Hannes disputes the architectural premise, arguing the problem could be solved by improving Git’s native notes implementation or tracking review metadata externally.

The author has explicitly asked whether Hannes’ silence should be interpreted as a NACK, but no resolution has emerged. The series is technically complete (v3) with robust test coverage, but the core design question remains unresolved. No other reviewers have engaged with the thread.

---

### `includeIf worktree:<pattern>`: design concerns and integration failure
**Chen Linxuan**’s v7 series adding `includeIf worktree:<pattern>` and `worktree/i:<pattern>` to Git’s config system faces two hurdles. Patrick Steinhardt raised concerns about the complexity of the symlink-preserving path logic in `logical_path_from_cwd()`, which conflicts with his in-flight `ps/setup-split-discovery-and-setup` series. He proposed either removing the `chdir()` call during repository discovery (a larger architectural change) or documenting the limitation and deferring the fix.

Junio C Hamano reported that the series passes all tests standalone but fails `t1305` when merged into the `seen` integration branch. The conflicting topic is unidentified, and the author must bisect `seen` to isolate the interaction. The series is paused pending resolution of these issues.

---

### `the_repository` removal: integration conflicts
**Patrick Steinhardt**’s eight-patch series removing more uses of `the_repository` from the refs subsystem and worktree code is blocked by integration conflicts in `jch` and `seen`. The series systematically replaces implicit uses of `the_repository` with explicit `struct repository *` parameters across 25 files, including a latent bug fix for `core.packedRefsTimeout` (now per-repository via `struct packed_ref_store`).

Junio C Hamano identified a behavioral regression in the first patch: the new per-repository `timeout_value` field is zero-initialized, whereas the old static variable defaulted to 1000 ms. The fix is to explicitly initialize `timeout_value` to 1000 in `packed_ref_store_init()`. The series is otherwise uncontested but cannot progress until the integration conflicts are resolved.

---

## In brief

**Reftable compaction fix** -- Kristofer Karlsson’s v2 series fixing a quadratic performance regression in the reftable backend when tombstones are present is now in v3, with the `suppress_deletions` flag moved to `struct reftable_stack_options` for downstream compatibility.

**French translation update** -- Jean-Noël Avila’s ongoing documentation synopsis-style conversion effort continues, with no new patches today but steady progress in prior days.

**SSH signature inspection** -- Grayson Tinker followed up on a bugfix patch allowing SSH signature inspection without `gpg.ssh.allowedSignersFile`. The patch relaxes the requirement to show basic signature details while advising users to configure the file for full validation.

**`git-subtree` safeguard** -- Ian Jackson’s patch series adding a safeguard to `git-subtree` (in `contrib/`) to detect and reject repositories modified by a forthcoming Rust rewrite is now fully reviewed. All feedback has been addressed, and the series is ready for `next`.

**Commit-graph regression fix** -- Kristofer Karlsson’s two-patch bugfix for a performance regression in split commit-graph chains is now merged to `master`. The fix ensures the `topo_levels` slab is propagated to all layers of the chain, restoring original performance.

**HTTP daemon test helpers** -- Michael Montalbo’s three-patch series fixing race conditions in HTTP daemon test helpers is under review. The series replaces non-atomic shell operations with atomic `mv` and `mkdir` to prevent HTTP 500 errors in concurrent requests.

**Coverity fixes** -- Johannes Schindelin’s eleven-patch series addressing NULL-dereference issues identified by Coverity is queued for merging. Junio C Hamano identified a latent bug in the final patch (shallow.c) involving `oid_to_hex()`’s static buffer, which will be fixed in v3.

**64-bit Windows compatibility** -- Johannes Schindelin posted a twelve-patch series widening size-related fields in Git’s delta and pack machinery from `unsigned long` to `size_t` for 64-bit Windows compatibility. The series is uncontroversial and part of the broader `unsigned long` → `size_t` conversion effort.

**`git add -e` refactoring** -- Gatla Vishweshwar Reddy posted a patch refactoring `git add -e` to use Git’s internal `apply_all_patches()` API instead of forking a subprocess. The change is mechanical and confined to `builtin/add.c`.

---

## On the radar

- **`repo_config_values` migration** -- Tian Yuchen’s nine-patch series migrating global configuration variables into `struct repo_config_values` is now technically complete and memory-leak-free after addressing Junio C Hamano’s feedback. The series is ready for `next` pending further review.
- **`git replay --linearize` bug** -- A bug in v5 of the `git replay --linearize` series that silently drops commits when replaying a single branch with merges remains unresolved. The issue is internal to the unmerged series and does not affect released versions.