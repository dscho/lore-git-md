# The Git Project Daily Digest
**2026/06/28 (UTC)**

---

## The day in brief
A **moderately busy Sunday** with **45 emails across 17 threads**, dominated by **performance optimizations**, **regression fixes**, and **final polish** on long-running series. The standout developments: **Tian Yuchen’s `paint_down_to_common()` optimization** received final approval, **Harald Nordgren’s `git history squash` series** reached v6 with stricter input validation, and **Johannes Schindelin’s regression report** on `git replay --linearize` exposed a critical flaw requiring urgent follow-up. A **CI/platform-compatibility series** addressing macOS hangs in `t5551`/`t5559` also landed, resolving a persistent flake.

---

## Notable threads

### `paint_down_to_common()` optimization receives final approval
**Thread**: [commit-reach: optimize `paint_down_to_common()` for one-sided histories](https://lore.kernel.org/git/20260628122538.12345-1-kristofer.karlsson@gitlab.com/)
**Author**: Kristofer Karlsson (via Tian Yuchen)

Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories received **final approval from Derrick Stolee**, clearing the way for merging. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Key improvements in v4:
- **Regression fix**: Restored the `min_generation` guard to the generation-monotonicity BUG assertion, addressing the test failure that caused Junio to eject the series from `seen`.
- **Code clarity**: Unified halt conditions in the `paint_state` struct and renamed counters for consistency.
- **Instrumentation**: Added trace2 metrics for "commits walked" and deterministic step-count assertions for all commit-graph modes.

The optimization delivers **100-1000x speedups** for asymmetric queries (e.g., `merge-base --all` across a 2.6M-commit monorepo drops from 3.67s to 5ms). The series is **low-risk** (internal to the commit-reach subsystem, no on-disk format changes) and **high-reward**, with no known regressions. Junio is expected to queue it for `next` shortly.

---

### `git history squash` reaches v6 with stricter input validation
**Thread**: [PATCH v6 0/4] replay: introduce --linearize option](https://lore.kernel.org/git/20260628082905.6789-1-haraldnordgren@gmail.com/)
**Author**: Harald Nordgren

Harald Nordgren posted **v6 of the `git history squash` series**, addressing all prior feedback and adding stricter input validation. The series folds a commit range into its oldest commit while preserving descendant history, offering a simpler alternative to `git rebase -i` for linearizing histories. Key changes in v6:
- **Input validation**: Now rejects single-commit ranges, empty ranges, and ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **Range resolution**: Supports multiple revision arguments (e.g., `@~3.. ^topic`) to exclude commits on other branches.
- **Documentation**: Expanded to clarify the merge limitation and `--reedit-message` behavior.

The series is **functionally complete** and ready for Junio’s final review. The only open question is whether `--reedit-message` (or `--edit`) should be the default, with Phillip Wood advocating for it as a commit hygiene measure. The implementation is **well-tested** (747-line test suite) and aligns with the project’s preference for explicit, predictable behavior.

---

### `git replay --linearize` regression requires urgent follow-up
**Thread**: [PATCH 0/3] replay: introduce --linearize option](https://lore.kernel.org/git/20260608183718.12345-1-toon@iotcl.com/)
**Author**: Johannes Schindelin

Johannes Schindelin reported a **regression in `git replay --linearize`** (merged in v5) that **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact behavior. Schindelin provided a concrete test case:
```sh
git replay --linearize --onto master~2 master~2..master
```
Expected: All commits in `master~2..master` are replayed.
Actual: Only the tip commit is replayed, omitting the first commit ("Git 2.55-rc2").

The regression is **urgent** and must be fixed before the next release. Toon Claes is expected to send a follow-up patch to restore the `replayed_base` logic or redesign how multi-branch histories are handled. The thread highlights the importance of **preserving algorithmic invariants** during refactoring, even in experimental commands like `git replay`.

---

### CI/platform-compatibility series lands
**Thread**: [PATCH 0/3] macOS CI hang in t5551/t5559 – root cause and fix](https://lore.kernel.org/git/20260628075716.12345-1-peff@peff.net/)
**Author**: Jeff King

Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` was **merged into `next`**. The series targets Apache bug 70131, which causes HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. Key changes:
- **Patch 1/3**: Increases Apache’s `Timeout` directive from 300 to 600 seconds in `t/lib-httpd/apache.conf`.
- **Patch 2/3**: Isolates the expensive "many-tags" test case into a dedicated repository, reducing runtime from ~95s to ~37s for EXPENSIVE runs.
- **Patch 3/3**: Packs refs after creating many tags, reducing `ls-refs` advertisement time from ~1.2s to 24ms.

The series is **low-risk** and resolves a persistent CI flake. Junio and Peff debated whether the third patch (ref-packing) was necessary, but it was included as a belt-and-braces measure. The root cause—Apache’s 300-second CGI timeout—is now **publicly documented** via an external pull request to Apache’s GitHub mirror.

---

### `excludes_file` libification guardrail debate resolved
**Thread**: [PATCH v2 0/2] Move excludes_file into repo_config_values](https://lore.kernel.org/git/20260628125835.12345-1-cat@malon.dev/)
**Author**: Tian Yuchen

Tian Yuchen and Junio C Hamano resolved the **guardrail debate** for the `excludes_file` libification series. The final design removes the redundant `repo->initialized` check from `repo_excludes_file()` and relies entirely on `repo_config_values()` to enforce initialization. The series is now **ready for `next`**, with CI failures expected (and intentional) as the stricter design surfaces previously hidden invalid callers. Key takeaways:
- **Centralized validation**: `repo_config_values()` is the canonical place for initialization checks.
- **Precedent**: Future repository-aware getters (e.g., `attributes_file`) will follow the same pattern.
- **Memory safety**: Heap allocation for `excludes_file` is now properly managed via `repo_config_values_clear()`.

The series is part of the broader `the_repository` removal effort and has no user-visible behavior changes.

---

### `USE_NSEC` Meson parity discussion expands
**Thread**: [PATCH] meson: add `nanosec` option to mirror Autotools `USE_NSEC`](https://lore.kernel.org/git/20260628081806.12345-1-peff@peff.net/)
**Author**: Jeff King

Jeff King’s testing expanded the `USE_NSEC` Meson parity discussion, confirming that **nanosecond timestamps are preserved correctly** on modern Linux filesystems (ext4, ext2, XFS, vfat, CIFS, NTFS, FUSE). His findings suggest the original "racy Git" concerns are obsolete, but he also identified **interoperability risks** when mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit). Key points:
- **Filesystem coverage**: Testing now includes XFS, bolstering the case for flipping the default to `true` on Linux.
- **Interoperability**: A non-`USE_NSEC` Git writing an index with millisecond timestamps would appear stat-dirty to an `USE_NSEC`-enabled Git, forcing unnecessary re-reads.
- **Runtime configuration**: Patrick Steinhardt proposed making `USE_NSEC` runtime-configurable (e.g., `core.useNsec`), avoiding auto-detection challenges.

The discussion now centers on whether to **deprecate the knob**, flip the default, or make it runtime-configurable. D. Ben Knoble’s original patch remains queued, but its justification is now in question.

---

## In brief

**`git branch --set-upstream-to` usability fix** -- Harald Nordgren’s **v3 series** improving error messages for common command-line mistakes (e.g., `git branch --set-upstream-to=origin main`) is **merged into `next`**. The series adds two new config options (`advice.setUpstreamFailure` and `advice.pushRepoLooksLikeRef`) and is **ready for `master`**.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged**. The patches remove `--statistics` from `msgfmt` invocations and conditionally suppress "Generating catalog" messages during `make -s`.

**`git history squash` refactoring** -- Harald Nordgren’s **v6 series** includes a **refactoring patch** extracting `first_parent_tree_oid()` to centralize logic for resolving a commit’s first-parent tree. The patch is uncontroversial and ready for merging.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **bugfix patch** for a regression rejecting HTTPS proxy URLs is **merged into `master`**. The fix adds a missing `return 0` in `set_curl_proxy_type()` and preserves the function’s contract of rejecting unsupported proxy schemes.

**Reftable memory leak fix** -- Jeff King’s **bugfix patch** for a memory leak in `reftable_writer_new` is **merged into `pu`**. The fix moves the allocation after input validation and resolves a minor textual conflict with a recent commit.

**French translation update** -- Jiang Xin’s **l10n pull request** for Git 2.55.0 is **ready for merging**. The update covers 11 languages and includes a mass typo-fix pass for French.

---

## On the radar

**`git replay --linearize` regression** -- Toon Claes must send a follow-up patch to fix the silent commit dropping issue. The regression is **urgent** and must be addressed before the next release.

**`git repack --geometric --cruft` RFC** -- Taylor Blau’s series needs a fix for the reachability filtering flaw. The series’ future may depend on Taylor’s availability after a role change.

**`git history squash`** -- Harald Nordgren’s **v6 series** is ready for Junio’s final review. The only open question is whether `--reedit-message` should be the default.

**ODB abstraction** -- Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review. The series is foundational for pluggable ODB backends.

**`--track=fetch` debate** -- Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on workflow concerns. Junio’s reservations about network operations during checkout without user preview persist.