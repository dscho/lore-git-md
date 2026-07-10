# The Git Mailing List Daily Digest for 2026/07/09

**The day in brief**
A busy Thursday with 108 emails across 23 threads. The standout developments: a **performance regression fix** for the reftable backend lands in v3, **ODB abstraction work** advances with Junio’s praise for a clean refactoring, and **new worktree hooks** debut to support AI-driven development environments. Architectural debates continue on `includeIf worktree:` and external notes commands, while routine refactoring and bugfixes fill out the day.

---

## Notable threads

### **Reftable backend performance regression fixed (v3)**
**Thread**: [PATCH v3 0/2] reftable: fix quadratic performance with tombstones
**Author**: Kristofer Karlsson

The reftable backend’s quadratic performance regression—where operations like `refs_verify_refnames_available()` scanned all tombstones before bounds checks—is now resolved. The v3 series moves the `suppress_deletions` flag to `struct reftable_stack_options`, allowing downstream users (like libgit2) to control tombstone suppression at stack creation. Performance improves from ~14s to ~0.2s in the perf test’s 8000-ref scenarios. The fix is narrowly scoped, well-motivated, and addresses prior feedback about compatibility and test isolation. **Status**: Ready for `next`.

---

### **ODB abstraction: `git cat-file` refactored to use filters**
**Thread**: [PATCH 0/7] Introduce object filters to `odb_for_each_object()`
**Author**: Patrick Steinhardt

Junio C Hamano praised the final patch in this series as "very nice," noting its 7-insertion/69-deletion diffstat that replaces backend-specific logic in `git cat-file --batch-all-objects` with ODB-mediated filtering. The series prepares the packed backend for pluggable ODBs by improving object lookup efficiency, enabling early termination of bitmapped object iteration, and introducing a `filter` field in `odb_for_each_object_options`. **Status**: Under review; minor commit-message tweaks requested.

---

### **New worktree hooks for AI-driven workflows**
**Thread**: [PATCH 0/3] Add `post-worktree-add` and `post-worktree-remove` hooks
**Author**: Domen Kožar

Two new hooks debut to support automated workflows (particularly AI-driven development environments) that create and destroy worktrees at high velocity. The `post-worktree-add` hook runs after `git worktree add` completes, even with `--no-checkout` or `--orphan`, while `post-worktree-remove` fires after explicit removal *or* pruning. The series is well-tested and addresses a real gap in Git’s automation story. **Status**: New; likely to spark discussion about hook semantics (e.g., whether `post-worktree-add` should run if `post-checkout` fails).

---

### **`includeIf worktree:` series hits integration snag**
**Thread**: [PATCH v7 0/3] config: add `includeIf worktree:<pattern>`
**Author**: Chen Linxuan

The v7 series, which adds `includeIf worktree:<pattern>` and `worktree/i:<pattern>` to Git’s config system, now faces two hurdles:
1. **Integration failure**: Junio reports the series breaks `t1305` when merged into `seen`, though it passes standalone. The conflicting topic is unidentified.
2. **Design concerns**: Patrick Steinhardt critiques the symlink-preserving path logic as "awkward" and notes it conflicts with his in-flight `ps/setup-*` series. He proposes either removing the `chdir()` call during repository discovery or documenting the limitation.
**Status**: Blocked; author must diagnose the `seen` conflict and address Patrick’s architectural feedback.

---

### **External notes commands: architectural limbo**
**Thread**: [PATCH v3 0/4] notes: add support for external notes commands
**Author**: Siddh Raman Pant

The author explicitly asks whether Johannes Sixt’s (Hannes) silence on the v3 series should be interpreted as a NACK. The thread remains in architectural limbo: Hannes disputes the premise (arguing the problem could be solved by improving Git’s native notes implementation or tracking review metadata externally), while the author defends the design as a pragmatic middle ground for TOCTOU issues in large repositories. **Status**: Stalled; no technical review of the implementation yet.

---

### **`git refs` subcommands: usability questions linger**
**Thread**: [PATCH v4 0/5] refs: add delete, update, create, and rename subcommands
**Author**: Patrick Steinhardt

The merged series consolidating reference manipulation under `git refs` now faces usability questions about the `--no-deref` option. Toon Claes proposes **eliminating `--no-deref` entirely**, arguing its behavior is too obscure and that users should instead use `git ref` (for dereferenced operations) or `git symbolic-ref` (for symref-specific operations). The discussion highlights a tension between atomicity guarantees and CLI ergonomics. **Status**: Merged to `master`; follow-up patches likely.

---

## In brief

**Bugfixes and refactoring**:
- **Commit-graph regression**: Kristofer Karlsson’s two-patch fix for `topo_levels` slab propagation in split commit-graph chains is **merged to `master`**.
- **Coverity fixes**: Johannes Schindelin’s eleven-patch series plugging NULL-dereference holes (e.g., in `bisect.c`, `reftable/stack.c`, `shallow.c`) is **under review**; Junio requests a v3 for a latent `oid_to_hex()` bug.
- **`the_repository` removal**: Patrick Steinhardt’s eight-patch series removing `the_repository` from the refs subsystem is **blocked on integration conflicts** in `jch`/`seen`.
- **64-bit Windows fixes**: Johannes Schindelin’s twelve-patch series widening size fields in the delta/pack machinery to `size_t` is **under review**; no objections yet.
- **Test modernization**: Gatla Vishweshwar Reddy’s patch replacing `git reflog | wc -l` with `test_stdout_line_count` is **queued for `next`**.

**New patches**:
- **`git add -e` refactoring**: Gatla Vishweshwar Reddy replaces a subprocess call to `git apply` with the internal `apply_all_patches()` API. **Status**: New; uncontroversial.
- **SSH signature inspection**: Grayson Tinker’s patch relaxes the requirement for `gpg.ssh.allowedSignersFile` to display SSH signature details. **Status**: New; no objections yet.

**Documentation and usability**:
- **`git fetch` inconsistency**: Ondřej Medek reports that `git fetch`’s automatic tag fetching depends on whether any refspec includes a `:dst` part, contradicting the documentation. **Status**: Reported; no resolution yet.

---

## On the radar

- **`repo_config_values` migration**: Tian Yuchen’s nine-patch series migrating global config variables into `struct repo_config_values` is **technically complete** after fixing a memory leak in `core_pager_config()`. **Status**: Ready for `next`; RFC questions about program-path migration remain open.
- **`git-subtree` safeguard**: Ian Jackson’s patch adding a safeguard against mixing old and new `git-subtree` implementations is **under review**; all feedback addressed. **Status**: Likely to land soon.