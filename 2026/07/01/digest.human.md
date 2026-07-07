# The Git Project Mailing List Daily Digest for 2026/07/01

## The day in brief

A busy day on the Git mailing list, with **134 emails across 29 threads** covering everything from security-hardened protocol extensions to build system tweaks and performance optimizations. The standout developments:

- **`git cat-file --batch-command` gains `remote-object-info` support** (Pablo Sabater's GSoC project), allowing clients to query object metadata from remotes without downloading full objects. The **15th iteration** is now ready for merging after addressing all review feedback, including a critical refactoring flaw identified by Junio Hamano.
- **`git history drop` reaches completion** after Patrick Steinhardt resolved a high-impact dispute over ref resolution logic. The 11-patch series is now technically complete and carries Junio's explicit approval.
- **Junio's "What's cooking" report** provides a comprehensive snapshot of the 48 in-flight topics at the start of the Git 2.56 cycle, with three ODB/ref-related topics now explicitly marked as ready for `next`.

The day also saw progress on long-running efforts like the **`lib/` directory reorganization**, **reftable hardening**, and **commit-reach optimizations**, alongside targeted bugfixes and CI improvements.

---

## Notable threads

### `git cat-file --batch-command` gains `remote-object-info` support
**Headline**: Security-hardened remote object metadata queries now ready for merging

Pablo Sabater's **15th iteration** of the `remote-object-info` feature for `git cat-file --batch-command` is now complete and ready for merging. The series allows clients to query object metadata (initially just size) from remote repositories without downloading full objects, using a new `remote-object-info` command that works with protocol v2 servers advertising the `object-info` capability.

### Key improvements in v15

- **Dynamic capability-based validation**: Runtime filtering of format placeholders against server-advertised capabilities
- **Platform-independent numeric conversion**: Added `strtoumax_szt()` helper to ensure consistent `size_t` handling across 32-bit and 64-bit platforms
- **Memory safety**: Removed unnecessary `static` declarations, added unconditional cleanup, and fixed a transport helper leak
- **Test infrastructure**: 680 lines of new tests in `t/t1017-cat-file-remote-object-info.sh`

**Critical fix**: Junio Hamano identified a refactoring flaw in patch 5/13 (uninitialized local variable and lingering global variable in `write_fetch_command_and_capabilities()`), which Pablo addressed by initializing the local variable to `0` and splitting the unrelated `hash_algo` type change into a separate patch.

### Security considerations

- Strict protocol v2 enforcement
- 10,000-object batch limit to prevent DoS
- Input validation for malformed OIDs and short OIDs
- Silent continuation for unsupported fields (matching `for-each-ref` behavior)

**Status**: Ready for merging. All prior feedback has been addressed, including Junio's critical refactoring fix, Karthik Nayak's protocol compatibility concerns, and Chandra Pratap's memory management suggestions.

---

### `git history drop` reaches completion
**Headline**: Commit-dropping subcommand now technically complete after dispute resolution

Patrick Steinhardt's **11-patch series** introducing the `git history drop` subcommand is now complete after resolving a high-impact dispute over ref resolution logic in `find_head_tree_change()`. The series allows users to remove a commit from history and replay its descendants on top of its parent, with conflict detection, bare repository support, and preservation of local changes.

### Key technical details

- **Ref resolution fix**: Added `RESOLVE_REF_READING` to `refs_resolve_ref_unsafe()` to ensure correct HEAD movement detection in detached-HEAD states
- **Reset API modernization**: Renamed `reset_head()` to `reset_working_tree()`, converted flags to an enum, added dry-run mode, and made HEAD updates opt-in
- **Ref update phases**: Split `handle_reference_updates()` into `compute_pending_ref_updates()` and `apply_pending_ref_updates()` for conflict detection
- **Test coverage**: 561 lines of new tests in `t/t3454-history-drop.sh`

**Dispute resolution**: Junio initially identified a mismatch between the ref names stored in `result->updates[]` and the symbolic branch name that `find_head_tree_change()` searches for. Patrick disagreed with Junio's analysis but ultimately implemented a fix that added `RESOLVE_REF_READING` and removed redundant code, which Junio accepted.

**Status**: Technically complete and carrying Junio's explicit approval. The series advances several ongoing efforts: `the_repository` removal, reset API modernization, and the experimental `git history` command.

---

### Junio's "What's cooking" report
**Headline**: 48 topics in flight at the start of Git 2.56 cycle

Junio Hamano's comprehensive report provides a snapshot of all in-flight topics at the start of the Git 2.56 cycle. Key highlights:

### New topics (7)

- **`kk/commit-reach-find-all-fix`**: Fixes `paint_down_to_common()` early-exit bug in v1 commit graphs with clock skew
- **`ps/setup-split-discovery-and-setup`**: Major refactoring of `setup.c` to split repository discovery and configuration into two phases
- **`pw/rebase-drop-notes-with-commit`**: Corrects rebase notes-copying logic for dropped commits (15 commits, expecting reroll)

### Stalled topics (4)

- **`ap/http-redirect-wwwauth-fix`**: Preserves WWW-Authenticate headers across redirects (stalled since June 2)
- **`js/parseopt-subcommand-autocorrection`**: Adds subcommand autocorrection to `git-remote` and `git-notes` (11 commits, stalled since April 27)

### Cooking topics (37)

- **`tb/repack-geometric-cruft`**: Teaches `--geometric --cruft` to roll up non-cruft packs (11 commits, expecting reroll)
- **`ps/odb-drop-whence`**: Removes `whence` from `struct object_info` (7 commits, **ready for `next`**)
- **`ps/reftable-hardening`**: Hardens reftable backend against corruption (12 commits, includes new fuzzer)
- **`ps/cat-file-remote-object-info`**: Adds `remote-object-info` to `git cat-file --batch-command` (13 commits)
- **`ps/history-drop`**: Adds `git history drop` subcommand (11 commits)
- **`ps/refs-writing-subcommands`**: Adds `create`, `delete`, `update`, `rename` to `git refs` (5 commits, **ready for `next`**)

### ODB/ref topics ready for `next`

- **`ps/odb-drop-whence`**: Ready after minor reroll (renaming `sourcep` to `source_infop`)
- **`ps/odb-generalize-prepare`**: Ready with no further changes
- **`ps/refs-writing-subcommands`**: Ready with no further changes

**Status**: The report provides the authoritative reference for what's queued for the next release and what needs review or reroll. Three ODB/ref-related topics are now explicitly marked as ready for `next`.

---

### `git replay --linearize` interface design debate
**Headline**: `--linearize` flag remains standalone despite consistency concerns

Toon Claes's **v6 series** introducing the `--linearize` option to `git replay` continues to use a standalone flag rather than adopting `git rebase`'s `--rebase-merges=<mode>` syntax. The series flattens merge commits to produce a linear history, with a predictable, all-or-nothing behavior that avoids dangling commits.

### Key debate points

- **Johannes Schindelin** strongly opposes mirroring `git rebase`'s syntax, calling it user-hostile and advocating for distinct option names that reflect the "replay" concept
- **Patrick Steinhardt** initially proposed adopting `git rebase`'s syntax for consistency but accepted the divergence with explicit justification
- **Junio Hamano** clarified the behavioral difference between `git replay --linearize` and `git rebase --no-rebase-merges`, which is now documented and tested

**Status**: The series is technically complete and ready for review. The interface design debate is no longer blocking, but the explicit justification for the divergence remains a point of discussion.

---

## In brief

**Rustification build system adjustment** -- Shardul Natu and Koji Nakamaru refined the integration of Rust code into Git's build system, focusing on macOS Universal Binary support for Rust components and fixing a parallel build race condition. The **v4 series is finalized and ready for integration**.

**`git history squash`** -- Harald Nordgren's **v7 series** implementing the `git history squash` subcommand is now complete, with all prior feedback addressed. The series allows users to collapse a linear or merge-commit-containing range into its oldest commit while preserving descendant history.

**Commit-reach optimizations** -- Tian Yuchen and Kristofer Karlsson's **v6 series** optimizing `paint_down_to_common()` for one-sided histories is now fully reviewed and ready for merging. The series delivers **100-1000x speedups** for asymmetric queries by terminating early when one side of the history is exhausted.

**Coverity fixes** -- Johannes Schindelin's **13-patch series** addressing Coverity-identified resource leaks and error-path bugs is now complete and carries Junio's explicit approval. The series plugs memory, file-descriptor, and process-handle leaks across core Git.

**Test modernization** -- Bryan B. Lima's patch modernizing `t7412-submodule-absorbgitdirs.sh` to use descriptive helper functions (`test_path_is_file`, `test_path_is_dir`) is ready for merging after addressing a minor `Signed-off-by` trailer order issue.

**HTTP/HTTPS authentication regression** -- A user reported that HTTP/HTTPS authentication using domain account passwords broke in Git for Windows 2.55.0.windows.1. Johannes Schindelin redirected the discussion to the Git for Windows issue tracker ([#6308](https://github.com/git-for-windows/git/issues/6308)).

**`git gui` encoding fix** -- Martin Malec fixed an encoding mismatch in Git for Windows's `git gui` that prevented it from starting when the user's home directory contained non-ASCII characters.

**Meson build race fix** -- D. Ben Knoble fixed a build race in generating `hook-list.h` that caused failures when the header was missing during compilation of `builtin/bugreport.c`.

---

## On the radar

**`lib/` directory reorganization** -- Patrick Steinhardt's RFC series to reorganize Git's source tree by moving `libgit.a` components into a new `lib/` directory continues to generate discussion. The series aims to reduce root-directory clutter and establish a clearer boundary for Git's core library code, but faces pushback over workflow disruption and `git log --follow` disruption.

**Recoverability concerns in `git history squash`** -- Matt Hunter and Phillip Wood raised concerns about the recoverability of `git history squash` operations, particularly when `--update-refs` moves multiple branches. Junio clarified that the reflog was never designed for undo, and Patrick Steinhardt proposed an **oplog (operations log)** as a long-term architectural solution.

**Quadratic-time behavior in reftable backend** -- Kristofer Karlsson identified and fixed a quadratic-time behavior in the reftable backend during bulk ref deletion and re-creation. The fix exposes tombstones to iterator bounds checks, reducing runtime from O(n²) to O(n). The patch's real-world impact remains under discussion.