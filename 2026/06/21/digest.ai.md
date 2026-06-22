# The Git Project Mailing List Daily Digest

**2026/06/21 in brief.** A Sunday with moderate traffic (56 emails, 15 threads) that delivered several long-running series to completion. The standout was **Taylor Blau's `--delta-islands` + `--path-walk` integration**, which graduated to merge-ready status after empirical validation showed it preserves path-walk's compression benefits while delivering 99% speedups. Two GSoC series also reached the finish line: **Pablo Sabater's `git cat-file --batch-command` remote-object-info** (13 iterations) and **Harald Nordgren's `git branch --delete-merged`** (16 iterations), both now cleared for merging. A quiet build-system patch added Meson parity for `USE_NSEC`, while a performance investigation into worktree diffs produced a minimal fix that unconditionally enables nanosecond-precision timestamp comparisons.

---

## Notable threads

### `git pack-objects`: `--delta-islands` + `--path-walk` integration (merge-ready)
**Taylor Blau's 4-patch v3 series** resolves the long-standing limitation where `--path-walk` could not leverage bitmaps or delta-islands. The implementation enables sequential compatibility: bitmaps accelerate object counting at the reachability layer, while path-walk handles delta compression at the pack-objects layer. Empirical validation across multiple repositories (including fluentui) shows dramatic speedups (up to 99.3% reduction in server time) while preserving path-walk's compression benefits (e.g., 558.4M → 164.4M in fluentui). The series updates the p5311 perf test to force fresh repacks, ensuring accurate measurement of `--path-walk`'s impact. Derrick Stolee's earlier request for validation is now satisfied, and Junio has queued the series for `next`.

### `git cat-file --batch-command` remote-object-info (GSoC, merge-ready)
**Pablo Sabater's 13-iteration security-hardened series** implements full client-server protocol support for querying object metadata from remotes without full downloads. The final v13 resolves all technical and design issues except one confirmed memory management concern (unnecessary `static` declarations in `parse_cmd_remote_object_info()`) and a documentation nit. Chandra Pratap's substantive review confirmed the design choices: clean failure for unsupported format atoms (replacing empty strings) and rejection of short OIDs for safety. Junio Hamano endorsed the memory management concern, leaving only mechanical fixes before merging. The series demonstrates thorough attention to protocol safety, dynamic capability validation, and cross-version compatibility.

### `git branch --delete-merged` (GSoC, merge-ready)
**Harald Nordgren's 16-iteration series** provides safe automated local branch cleanup with comprehensive safety mechanisms. The final design resolves the stacked-branch safety question by **aborting deletion** (with a clear error message) if a branch is used as an upstream for any other branch, avoiding the complexity of retargeting. Phillip Wood and Junio Hamano endorsed this simpler approach, which preserves backward compatibility for future retargeting enhancements. The implementation includes `--forked` filtering, per-branch opt-out via `branch.<name>.deleteMerged`, and `--dry-run` preview functionality. All technical feedback has been addressed, and the series is now ready for final review.

### `git log --graph` cascading indentation (v6, follow-up work required)
**Pablo Sabater's v6 series** implements cascading indentation for visual roots in `git log --graph`, resolving the architectural blocker from v5 through a clean abstraction layer in `revision.c`. Junio Hamano's latest review identified a forward-compatibility concern: the loop in `revision_has_commits_after()` directly accesses `prio_queue` internals, which may conflict with Kristofer Haugsbakk's ongoing `prio-queue` refactoring. Jeff King's analysis confirmed the abstraction layer needs iteration support (not just peeking) to handle history simplification scenarios. The series is technically complete but requires follow-up work for safety, forward-compatibility, and iteration support before the abstraction layer can be considered fully reviewed.

### Worktree diff performance (minimal fix proposed)
**Jeff King's investigation** into worktree diff slowdowns (223.3ms vs 3.4ms) identified timestamp precision as the root cause. His proposed 8-line patch removes the `#ifdef USE_NSEC` guard in `is_racy_stat()`, enabling nanosecond-aware timestamp comparisons unconditionally. Testing confirms this eliminates racy entries at the source, resolving the performance issue without architectural changes. Junio Hamano acknowledged the patch as "cute," signaling openness to the approach. The fix leverages existing infrastructure and avoids the performance tradeoffs of earlier proposals, though the original rationale for the `USE_NSEC` conditional remains unaddressed.

---

## In brief

**`git repo info` path formatting (GSoC, ready for pickup)** -- Lucas Seiki Oshiro's v7 series teaches `git repo info` to expose repository paths in both absolute and relative formats using a new `append_formatted_path()` helper. Junio Hamano's latest feedback suggests optional stylistic improvements to the helper's API clarity, but the series is functionally complete and ready for pickup.

**Ref backend refactoring (v4, architectural feedback)** -- Patrick Steinhardt's 10-part series modernizing Git's reference backend infrastructure received substantive review from Jeff King. The discussion shifted from a narrow recursion fix to a broader question about how ref backends should interact with the config system, particularly for early config reads that may skip includes. Peff's feedback will likely influence the final design of the `ignore_refs` flag introduced in patch 9/10.

**`git history squash` (v4, code-complete)** -- Harald Nordgren's 4-patch series implementing `git history squash <range>` is now code-complete, with all prior technical and design concerns resolved. The final design rejects operations with refs pointing inside the squashed range by default, advising `--update-refs=head` to explicitly retarget the current branch. The series is ready for Junio's final assessment.

**Shell completion dotfile handling (v3, structural overhaul required)** -- Zakariyah Ali's series to hide dotfiles by default in Git's path completion remains blocked on Junio Hamano's request for a cleaner two-patch progression: (1) refactor the AWK script for readability without changing behavior, and (2) implement dotfile hiding. The series will not proceed until this structural overhaul is addressed.

**Build system quietness (merged-ready)** -- Harald Nordgren's two-patch series aligning gitk and git-gui's translation catalog generation with core Git's quiet build conventions is now merged-ready. Johannes Sixt and Johannes Schindelin have effectively signed off on the gitk and git-gui portions, respectively.

**`ignore_case` libification (merged)** -- Justin Tobler and Tian Yuchen's two-patch series moving the global `ignore_case` variable into `struct repo_config_values` was accepted by Junio Hamano with "Looks good." The series eliminates a global and ties the configuration value to the repository instance it was read from, continuing the `the_repository` removal initiative.

**Meson `nanosec` option (queued)** -- D. Ben Knoble's patch adding a `nanosec` Meson option to mirror Autotools' `USE_NSEC` was accepted and queued by Junio Hamano. Jeff King raised a substantive caution about `USE_NSEC`'s reliability on filesystems with coarse timestamp granularity, but the patch itself is uncontroversial and ensures build system parity.

---

## On the radar

**Pinned references (RFC, design phase)** -- Erik Östlund's proposal for "pinned references" (a ref + expected OID invariant) remains in early-stage design discussion. Jeff King and Junio Hamano ruled out modifying the existing `<describeOutput>` syntax due to backward-compatibility constraints, shifting the discussion toward alternative designs (e.g., a new command or option). No code or patches exist yet.

**macOS CI hang fix (under review)** -- Michael Montalbo's proposed fix for macOS CI hangs in `t5551` and `t5559` received substantive review from Jeff King, who raised concerns that the fix may mask rather than eliminate the underlying HTTP/2 deadlock. The patch remains under review pending environment version checks.