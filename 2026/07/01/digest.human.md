# The Git Project Mailing List Daily Digest for 2026/07/01

## The day in brief

A busy day on the Git mailing list—134 emails across 29 threads—with **two major feature series reaching completion**, several **long-running refactoring efforts making final progress**, and a **handful of critical bugfixes** landing. The standout developments: **Patrick Steinhardt’s `git history drop` series is now fully merged** after resolving a critical ref-resolution dispute, and **Pablo Sabater’s `git cat-file --batch-command` remote-object-info feature is ready for integration** pending one last fix. Meanwhile, **Junio C Hamano’s "What’s cooking" report** provides a comprehensive snapshot of the 2.56 cycle, highlighting 48 in-flight topics and clarifying the status of several ODB/ref-related efforts.

---

## Notable threads

### `git history drop` lands after critical dispute resolution
**Topic**: `ps/history-drop` (11 patches)
**Author**: Patrick Steinhardt
**Status**: **Merged to `next`**

Patrick Steinhardt’s long-running effort to add a `drop` subcommand to the experimental `git history` command reached a major milestone today. The series, which removes a commit and replays its descendants, was **blocked for a week** by a **critical ref-resolution dispute** between Patrick and Junio C Hamano. The issue centered on whether `find_head_tree_change()` was correctly identifying HEAD movements when `--update-refs=head` was used in detached-HEAD states. Junio’s **v8 iteration** resolved the dispute by adding `RESOLVE_REF_READING` to `refs_resolve_ref_unsafe()`, ensuring the function returns NULL when HEAD cannot be resolved rather than a dangling pointer. The fix was validated with a new test case and is now **fully merged**.

The series also advances several ongoing efforts:
- **Reset API modernization**: Renamed `reset_head()` to `reset_working_tree()`, converted flags to an enum, added dry-run mode, and made HEAD updates opt-in.
- **Ref-update phase separation**: Split `handle_reference_updates()` into computation and application phases to support conflict detection.
- **Code deduplication**: Exposed `replay_result_queue_update()` for reuse between replay and history code paths.

With **561 lines of new test coverage** and thorough edge-case handling (bare repositories, conflict detection, local change preservation), this is a **significant addition to Git’s history-editing toolkit**.

---

### `git cat-file --batch-command` remote-object-info ready for merge
**Topic**: `ps/cat-file-remote-object-info` (13 patches)
**Author**: Pablo Sabater
**Status**: **Ready for `next` pending one fix**

Pablo Sabater’s **security-hardened** feature series, which adds `remote-object-info` support to `git cat-file --batch-command`, is now **technically complete** after 15 iterations. The feature allows clients to query object metadata (initially just size) from remote repositories without downloading full objects, using a new protocol v2 capability (`object-info=size`). Key security measures include:
- **Dynamic capability-based validation**: The client filters requested format placeholders against what the server advertises, preventing information leaks.
- **Strict protocol enforcement**: 10,000-object batch limit, full object ID requirements, and silent continuation for unsupported fields.
- **Memory safety**: Fixed a critical refactoring flaw in patch 5/13 (uninitialized local variable and lingering global variable) identified by Junio.

The series includes **680 lines of new tests** and has been **extensively reviewed** by Karthik Nayak, Chandra Pratap, and Junio. The only remaining blocker is **Junio’s requested fix for patch 5/13**, which Pablo has confirmed is straightforward. Once addressed, the series is **ready for integration**.

---

### Source tree reorganization debate continues
**Topic**: `ps/libgit-in-subdir` (3 patches)
**Author**: Patrick Steinhardt
**Status**: **Awaiting consensus**

Patrick Steinhardt’s **RFC v3** to reorganize Git’s source tree by moving `libgit.a` components into a new `lib/` directory remains **contentious**. The series aims to reduce root-directory clutter (from ~550 to ~80 files) and enforce clearer coding conventions (e.g., avoiding `the_repository` or `die()` in library code). However, **Junio C Hamano and Phillip Wood** continue to raise **substantive concerns** about the **recurring mental overhead** of remembering file moves (e.g., `wt-status.c` → `lib/wt-status.c`) and the **disruption to `git log --follow`**.

Today’s discussion introduced a **potential middle ground**: Kaartic Sivaraam proposed adding a hint to `git show` when a file appears to have been renamed (e.g., suggesting `lib/wt-status.c` instead of `wt-status.c`). While this doesn’t resolve the core debate, it offers a **tooling-based mitigation** for the workflow disruption. The thread remains **polarized**, with proponents (Patrick, Oswald Buddenhagen) arguing for long-term maintainability and opponents (Junio, Phillip) emphasizing short-term friction.

---

### Coverity-driven leak fixes land
**Topic**: `js/coverity-fixes` (12 patches)
**Author**: Johannes Schindelin
**Status**: **Queued for `next`**

Johannes Schindelin’s **12-patch series** addressing Coverity-flagged resource leaks and error-path bugs is now **queued for merging**. The series plugs leaks in:
- **Loose object handling** (`loose.c`): File descriptor leaks in `load_one_loose_object_map()` and `write_one_object()`.
- **Run-command API** (`run-command.c`): Process handle leaks in `start_command()`.
- **Submodule code** (`submodule.c`): Memory leaks in `get_superproject_working_tree()`.
- **Windows-specific code** (`compat/mingw.c`): Handle leaks in process termination logic.

Junio’s **final review** called the series "a delight to read" and confirmed it will be queued. The patches are **mechanical, low-risk, and self-contained**, making them ideal for early-cycle integration.

---

### Bloom filter leak fixes
**Topic**: `jk/bloom-leak-fixes` (3 patches)
**Author**: Jeff King
**Status**: **Likely to merge to `next`**

Jeff King’s **three-patch series** fixes memory leaks in Git’s commit-graph Bloom-filter code, surfacing only when `GIT_TEST_COMMIT_GRAPH_CHANGED_PATHS=1` is set. The leaks occur in:
1. **Bloom-filter slab initialization** (`bloom.c`): Redundant initialization without intervening `deinit_bloom_filters()`.
2. **Revision-walk keyvec handling** (`revision.c`): Overwritten keyvecs during multiple traversals of the same `rev_info` structure.
3. **Line-log range-copy logic** (`line-log.c`): Unnecessary copies of `struct line_log_data`.

The series has received **unanimous approval** from Patrick Steinhardt, Derrick Stolee, and Junio, who praised its **conservative design** (e.g., explicit freeing over reuse to avoid subtle bugs). With **no open questions**, this is **ready for `next`**.

---

## In brief

### Build system and CI
- **Rustification**: Shardul Natu and Koji Nakamaru’s **macOS Universal Binary support** for Rust components is now **ready for integration** after Junio’s sign-off. The series resolves a parallel build race condition and adds `RUST_TARGETS` environment variable support.
- **CI Docker limits**: Johannes Schindelin’s patch to **raise PID limits in Dockerized CI jobs** for private GitHub repositories is **queued for `next`**. The fix addresses "cannot create async thread" errors in the t5xxx-t6xxx test range.
- **Meson build race**: D. Ben Knoble’s fix for a **build race in generating `hook-list.h`** is **ready for merging**, resolving a regression introduced by commit `2eb541e8f2a9`.

### Bugfixes
- **`git blame -b` output formatting**: René Scharfe’s patch to **align commit hash lengths with `core.abbrev`** is **ready for integration**, fixing a usability friction in manual workflows (e.g., `git rebase -i`).
- **HTTP/2 stall fix**: The **Apache `mod_http2` bug 70131** causing CI hangs in `t5551`/`t5559` is now **publicly documented** via an external pull request to Apache’s GitHub mirror, ensuring future contributors can access the diagnostic context.
- **Reftable quadratic-time behavior**: Kristofer Karlsson’s **fix for O(n²) runtime** in bulk ref deletion/re-creation is **under discussion**, with Patrick Steinhardt confirming the tombstone-based root cause. The patch reduces runtime from **14.93s to 0.145s** for 8,000 refs but adds complexity at call sites.

### Test modernization
- **`t7412-submodule-absorbgitdirs.sh`**: Bryan B. Lima’s patch to **replace raw test calls with `test_path_is_file`/`test_path_is_dir`** is **queued for `next`**, part of the ongoing community-wide effort to modernize test assertions.

### Documentation
- **`MyFirstContribution`**: William Yeh’s update to **recommend `b4` for patch submission** is **merged to `next`**, reflecting the project’s growing adoption of the tool.

---

## On the radar

### `git replay --linearize` interface debate
**Topic**: `tc/replay-linearize` (3 patches)
**Author**: Toon Claes
**Status**: **Awaiting consensus**

Toon Claes’s **`--linearize` option for `git replay`** remains **blocked on an interface design debate**. The series flattens merge commits into a linear history, but **Johannes Schindelin and Toon** argue for a standalone `--linearize` flag, while **Patrick Steinhardt** prefers mirroring `git rebase`’s `--rebase-merges=<mode>` syntax. The **commit message now includes an explicit justification** for the divergence, but the debate may resurface during review.

### `git history squash` default behavior
**Topic**: `hn/history-squash` (4 patches)
**Author**: Harald Nordgren
**Status**: **Awaiting decision**

Harald Nordgren’s **`git history squash`** subcommand, which folds a range of commits into one, is **ready for review** but faces a **usability question**: should `--reedit-message` (or `--edit`) be the default? **Phillip Wood** argues for it as a **commit hygiene measure**, while the current implementation requires explicit opt-in. The discussion highlights a **philosophical difference** about whether Git should encourage or enforce message review.

### Reflog recoverability critique
**Topic**: `pw/rebase-drop-notes-with-commit` (15 patches)
**Author**: Phillip Wood
**Status**: **Awaiting architectural discussion**

Phillip Wood’s **rebase notes-copying fix** exposed a **systemic recoverability gap**: `git reset --hard` is insufficient to undo operations affecting multiple refs, as Git’s reflog lacks visibility into which refs were modified. **Junio clarified** that the reflog was never designed for undo, and **Patrick Steinhardt proposed an oplog (operations log)** as a long-term solution. The critique remains **non-blocking** but may inspire future architectural work.

---

## Editorial note

Today’s traffic reflects a **maturing phase in the 2.56 cycle**, with several **long-running efforts reaching completion** and **new features landing**. The **`git history` and `git cat-file` series** are particularly notable for their **security-conscious designs** and **thorough review processes**, while the **source tree reorganization debate** underscores the **tension between long-term maintainability and short-term disruption**. The **Coverity and Bloom filter leak fixes** demonstrate the project’s **commitment to memory safety**, a recurring theme in recent months. With **48 topics cooking** and **Junio’s "What’s cooking" report** providing clear guidance, the next few weeks are likely to see **rapid integration of ready topics** and **focused discussion on the remaining contentious ones**.