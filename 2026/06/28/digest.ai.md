# The Git Project Daily Digest
**2026/06/28 (UTC)**

A **moderately busy Sunday** with **45 emails across 17 threads**, dominated by **performance optimizations**, **regression fixes**, and **final polish** on long-running series. The standout developments: **Tian Yuchen’s `paint_down_to_common()` optimization** received final approval, **Harald Nordgren’s `git history squash` series** reached feature-complete status, and **Johannes Schindelin uncovered a critical regression** in `git replay --linearize`. The day also saw **CI fixes**, **translation updates**, and **build system debates** reach resolution.

---

## The day in brief
**Sunday’s traffic was technical and forward-looking**, with **performance patches landing**, **regressions surfacing**, and **long-running series wrapping up**. The **one must-read thread**: Johannes Schindelin’s report of a **silent commit-dropping regression** in `git replay --linearize`—this will require urgent follow-up. The **one must-watch series**: Tian Yuchen’s **8-patch optimization of `paint_down_to_common()`**, now fully approved and delivering **100-1000x speedups** for asymmetric histories.

---

## Notable threads

### `git replay --linearize` regression: silent commit dropping
**Subject**: Re: [PATCH 0/3] replay: introduce --linearize option
**Author**: Johannes Schindelin
**Status**: **Urgent regression identified**; follow-up patch expected.

Johannes Schindelin reported a **critical regression** in `git replay --linearize` (merged in v5): when replaying a single branch containing merge commits (e.g., `master~2..master` with `--linearize --onto master~2`), the command **silently drops commits** instead of flattening the entire range. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent exactly this behavior. Schindelin’s test case (`master~2..master` with `--linearize --onto master~2`) now replays only the tip commit ("Git 2.55-rc2") instead of the full range ("Git 2.54" and "Git 2.55-rc2").

The regression is **urgent** and must be fixed before the next release. Toon Claes is expected to send a follow-up patch to restore the `replayed_base` logic or redesign how multi-branch histories are handled. The thread also surfaced a **design tension**: whether `--linearize` should flatten all replayed commits into a single topology (current behavior) or allow branch-separate replay (which risks commit duplication). Schindelin argues the current behavior is correct and predictable, while the regression demonstrates the need for careful handling of merge commits.

**Key technical details**:
- **Files**: `replay.c` (`pick_regular_commit()`, `replay_revisions()`).
- **Root cause**: Removal of `replayed_base` parameter in v5.
- **Test case**: `git replay master~2..master --linearize --onto master~2`.
- **Expected output**: All commits flattened into a single topology.
- **Actual output**: Only the tip commit is replayed.
- **Follow-up**: Restore `replayed_base` or redesign merge handling.

---

### `paint_down_to_common()` optimization: 100-1000x speedups approved
**Subject**: [PATCH v4 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories
**Author**: Tian Yuchen
**Status**: **Fully approved and ready to merge**; delivers **100-1000x speedups** for asymmetric histories.

Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories received **final approval** from Derrick Stolee. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts or shallow histories). Key improvements:
- **Regression fix**: Addressed an unconditional BUG assertion in patch 7/8 (now guarded by `min_generation`).
- **Code clarity**: Unified halt conditions in the `paint_state` struct.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).
- **Performance**: **100-1000x speedups** for asymmetric queries (e.g., `merge-base --all` across a 2.6M-commit monorepo drops from **3.67s to 5ms**).

The series is **low-risk** (internal to the commit-reach subsystem) and **high-reward** (dramatic speedups for common asymmetric queries). It is now **ready for Junio to merge** into `next`.

**Key technical details**:
- **Files**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, test scripts.
- **New symbols**: `paint_state` struct (with counters `parent1_count`, `parent2_count`, `mb_candidate_count`).
- **Behavior change**: Early termination when `(!state->parent1_count || !state->parent2_count) && !state->mb_candidate_count`.
- **Test coverage**: Edge cases (self, duplicate-twos, pending-stale, infinity-both-sides), deterministic step-count assertions.

---

### `git history squash` reaches feature-complete status
**Subject**: [PATCH v6 0/4] git history squash: fold commit ranges into a single commit
**Author**: Harald Nordgren
**Status**: **Feature-complete**; ready for Junio’s final review.

Harald Nordgren’s **4-patch series** implementing `git history squash` reached **feature-complete status** in v6. The command folds a commit range into its oldest commit while preserving descendant history, offering a simpler alternative to `git rebase -i` for linearizing histories. Key improvements in v6:
- **Input validation**: Rejects single-commit ranges, empty ranges, and ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`.
- **Multiple revision arguments**: Supports compound ranges (e.g., `@~3.. ^topic`).
- **Ref safety**: Rejects operations where any ref points to a commit inside the squashed range (with advice to use `--update-refs=head`).
- **`--reedit-message`**: Seeds the editor with all folded-in commit messages (oldest first), matching `git rebase -i`’s squash behavior.

The series is **well-tested** (747-line test suite) and addresses all prior feedback, including **merge commit handling** (rejects merges with external parents, allows fully contained merges) and **CLI/documentation style**. The only remaining debate is whether `--reedit-message` (or `--edit`) should be the default, but this is not blocking.

**Key technical details**:
- **Files**: `builtin/history.c`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
- **New helpers**: `resolve_squash_range()`, `reject_fixupish_oldest()`, `find_interior_ref()`, `build_squash_message()`.
- **Behavior**: Single three-way merge of the range against the oldest commit’s parent, followed by descendant replay.
- **Test coverage**: Range parsing, merge handling, ref safety, `--reedit-message` template format.

---

### CI hangs in `t5551`/`t5559`: Apache timeout fix merged
**Subject**: [PATCH 0/3] t/lib-httpd: bump apache timeout, isolate many-tags test
**Author**: Jeff King
**Status**: **Merged**; resolves macOS CI hangs.

Jeff King’s **3-patch series** addressing CI hangs in `t5551` and `t5559` was **merged into `master`**. The series:
1. **Increases Apache’s `Timeout` directive** from 300 to 600 seconds (resolving the root cause of HTTP/2 stalls and `curl 18` mid-transfer aborts).
2. **Isolates the expensive "many-tags" test case** into a dedicated repository, reducing runtime for the entire `t5551` script.
3. **Packs refs after creating many tags**, reducing `ls-refs` advertisement time from ~1.2 seconds to 24 ms.

The series is **low-risk** and directly addresses the root cause (Apache bug 70131). Junio noted a minor stylistic nit (non-bare repository initialization in patch 2/3), but the fix is now **integrated**.

**Key technical details**:
- **Files**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`.
- **Behavior change**: Apache waits longer for slow `upload-pack` responses; expensive test case runs faster.
- **Root cause**: Apache’s 300-second CGI timeout being hit during `ls-refs` advertisements of 100,000 loose refs.

---

### `USE_NSEC` Meson parity: build system debate reaches impasse
**Subject**: Re: [PATCH] meson: add `nanosec` option to mirror Autotools `USE_NSEC`
**Author**: D. Ben Knoble, Jeff King, Patrick Steinhardt
**Status**: **Queued but controversial**; debate over knob’s viability and default behavior.

D. Ben Knoble’s patch adding a `nanosec` Meson option to mirror Autotools’ `USE_NSEC` is **queued for merging**, but the discussion has expanded into a **broader debate** about the knob’s necessity. Key points:
- **Jeff King’s testing**: Nanosecond timestamps are preserved correctly on modern Linux (ext4, ext2, XFS, vfat, CIFS, NTFS, FUSE), obsoleting historical "racy Git" concerns.
- **Interoperability risks**: Mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit) could cause performance degradation (stat-dirty re-reads).
- **Proposals**:
  - **Brian M. Carlson**: Flip the default to `true` for most users, with a config knob for edge cases.
  - **Patrick Steinhardt**: Always compile nanosecond support into Git (when the platform allows it) and expose it via a runtime config knob (e.g., `core.useNsec`), but retain the build-time default as `false`.
  - **D. Ben Knoble**: Proceed with the build-time option for parity, unless `USE_NSEC` is deprecated entirely.

The thread is at an **impasse**: the patch is technically valid, but its justification is now in question. Junio has queued it, but the broader discussion may lead to **follow-up work** (e.g., runtime configuration or deprecation).

**Key technical details**:
- **Files**: `meson_options.txt`, `meson.build`.
- **New option**: `nanosec` (boolean, default `false`).
- **Macro defined**: `USE_NSEC`.
- **Proposed runtime knob**: `core.useNsec` (boolean, default TBD).

---

## In brief

**`git cat-file --batch-command`** -- Pablo Sabater’s **14-patch GSoC series** is **technically complete** and ready for merging. The series introduces a `remote-object-info` command for querying object metadata from remotes without full downloads, with **dynamic format validation** and **680 lines of new tests**.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**.

**HTTPS proxy regression** -- Johannes Schindelin’s **bugfix patch** for a regression rejecting HTTPS proxy URLs was **merged into `master`**. The fix adds a missing `return 0` in `set_curl_proxy_type()`.

**Reftable memory leak** -- Jeff King’s **bugfix patch** for a memory leak in `reftable_writer_new` was **merged into `pu`**. The fix moves the allocation after input validation.

**French translation update** -- Jiang Xin’s **l10n pull request** for Git 2.55.0 updates `.po` files for 11 languages and is **ready for merging**.

**`greplint.pl` regression** -- Junio C Hamano identified a **critical oversight** in the `greplint.pl` series: the proposed `test_grep` enhancement is redundant because `test_grep` already checks for file existence. The discussion now centers on **debugging why the existing safeguard fails** in `t3420-rebase-autostash.sh`.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit-dropping issue.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw.
- **`git history squash`**: Harald Nordgren’s **v6 series** is ready for Junio’s final review.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.
- **`--track=fetch` debate**: Harald Nordgren’s feature remains stalled on workflow concerns.