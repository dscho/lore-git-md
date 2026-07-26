The day in brief.
Saturday, July 25, 2026 saw **47 emails across 14 threads**, a light but technically rich day.
The standout event: **Harald Nordgren’s `git branch --delete-merged` series landed in Junio’s tree**, capping a 23-iteration journey with a final mechanical fix.
Also notable: **a critical memory-safety bug in `git worktree add` surfaced**, drawing immediate review attention, and **Junio’s “What’s cooking” report** revealed 47 topics still in flight, including several stalled efforts needing rerolls.

---

### Notable threads

**`git branch --delete-merged` lands after 23 iterations**
Harald Nordgren’s long-running series finally cleared its last hurdle: Junio C Hamano’s review of patch 1/7 identified two mechanical issues (a type mismatch and a readability nit in `short_upstream_name()`), which Harald fixed in v23.
The series is now **queued in Junio’s tree** and will appear in the next “What’s cooking” report.
It introduces a **safe, automated local-branch cleanup command** with comprehensive safety controls—order-independent stacked-branch protection, per-branch opt-out via `branch.<name>.deleteMerged=false`, `--dry-run` preview, and repeatable upstream selectors.
The only agreed follow-up is extending stacked-branch protection to `git branch -d` for consistency, which Harald will implement post-merge.

---

**Critical memory-safety bug in `git worktree add`**
Matthias Aßhauer reported a **memory-safety bug in `worktree_basename()`** that can cause `git worktree add` to perform an out-of-bounds read when given an empty string or a string consisting solely of directory separators (e.g., `""` or `"//"`).
The bug can lead to **silent data corruption**—recursive deletion of the working directory, including `.git`—or trigger a BUG message (`How come '' becomes empty after sanitization?`).
Junio’s review of the proposed fix identified a **logical flaw**: the patch’s pointer arithmetic could still underflow, risking buffer allocation failures.
A v2 is expected to adopt Junio’s safer integer-index loop approach.
The issue was discovered in the wild (Git for Windows #6346), likely via unset environment variables in scripts.

---

**`git rebase --update-refs` symref bugfix series advances**
Son Luong Ngoc’s v3 series fixing `git rebase --update-refs` edge cases with symbolic references received substantive review from Erik Cervin-Edin.
Erik confirmed the v3 series resolves a regression in v2 (incorrect current-branch skipping due to `branch_checked_out()` behavior) but also surfaced a **pre-existing bug in `master`**: the `head_ref` buffer used for HEAD comparison is clobbered by `branch_checked_out()`, causing branches that should be skipped to incorrectly appear in the rebase todo list.
This latent bug explains unexpected todo entries in multi-worktree setups.
The v3 series is functionally complete, but Phillip Wood’s latest review requests an explanatory comment and minor test organization tweaks before v4.

---

**Packfile URI race-condition fix stalled on test flakiness**
Ted Nyman’s v4 series fixing packfile URI download race conditions remains **blocked** due to an intermittent `--stress` test failure in `t5550-http-fetch-dumb.sh`.
Jeff King (Peff) diagnosed the issue as a **false positive** caused by a race in the test’s `test -s` synchronization check, not a bug in the core logic.
Peff and Junio agree the `test -s` check should be dropped, as the deterministic staging path design makes the order of writes irrelevant.
The series is otherwise ready for `next`; a reroll with the test adjustment is expected soon.

---

### In brief

**`git cat-file --batch-command` gains `%(objecttype)` support**
Pablo Sabater’s GSoC series extends the recently merged remote-object-info functionality to include object type metadata.
The five-patch series adds end-to-end support: server capability advertisement, client request logic, protocol extension, and client-side parsing/display.
The implementation is mechanical, follows the established pattern from the predecessor series, and unifies the default format string for local and remote queries.

**`git checkout -m` autostash conflict handling refined**
Harald Nordgren posted a two-patch series that (1) makes the sequencer’s autostash apply logic explicitly report conflicts, and (2) skips the autostash retry loop when no tracked local changes exist.
The series also improves the visual separation of autostash conflict advice from the branch-switch confirmation message.

**`git merge-base --is-ancestor` test coverage added**
Nikolaus Schuetz added comprehensive test coverage for `git merge-base --is-ancestor` in `t/t6010-merge-base.sh`, which previously had none.
The 34-line patch verifies exit codes, error conditions, and edge cases (self-ancestry, diverged commits, non-existent commits).

**`the_repository` removal series hits architectural question**
Tian Yuchen’s three-patch series migrating four global configuration variables into `repo_config_values` drew a substantive review from Junio.
Junio objected to patch 1/3’s unconditional fallback to `the_repository` when `repo != the_repository`, arguing it papers over an API limitation.
He proposed `BUG()` as a forcing function to extend dependent APIs properly.
The series is otherwise mechanical and uncontroversial.

**Memory leaks plugged in `remote.c`**
Junio posted a v2 patch plugging memory leaks in `rewrites_release()`, which frees URL alias settings (`url.<real>.insteadOf` and `url.<real>.pushInsteadOf`).
Jeff King’s review confirmed the v2 approach (explicit string copying) is correct, clearing the patch for integration.

---

### On the radar

**`git repack --drop-filtered` RFC series**
Siddharth Shrimali’s RFC series for partial-clone disk-space reclamation settled its **safety guard requirements** and **CLI design** for v2.
The author confirmed that **merge/rebase/cherry-pick checks** and **index validation** (preventing drops of blobs referenced by the current index) will be implemented before posting v2.
The **drop log** is deferred to a follow-up, and a real `--drop-filtered` run will now **imply `-d`** (removing old promisor packs containing the dropped blobs).

**`git history squash` lands in `next`**
Han-Wen Nienhuys’s `git history squash` feature graduated to `next` in Junio’s latest “What’s cooking” report.
The command folds a range of commits into one, providing a more intuitive alternative to `git rebase -i` for certain workflows.

**ODB pluggability series needs review**
Patrick Steinhardt’s `ps/odb-make-creation-pluggable` (5 patches) and `ps/odb-pluggable-housekeeping` (12 patches) remain in `seen`, awaiting review.
These series are part of the long-term effort to make ODB backends pluggable.

**Stalled topics needing rerolls**
Junio’s report flagged several stalled topics as needing rerolls:
- `tb/repack-geometric-cruft` (unresolved feedback)
- `hn/checkout-track-fetch` (stalled for months)
- `za/completion-hide-dotfiles` (stalled)
- `kh/doc-trailers` and `kh/doc-replay-config` (both awaiting rerolls)