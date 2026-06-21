# The Git Project Mailing List Daily Digest - 2026/06/20

## The day in brief.

Saturday brought a moderate but technically rich day to the Git list, with 37 emails across 14 threads. The standout developments: Pablo Sabater’s `git log --graph` indentation series cleared its final architectural hurdle, Harald Nordgren’s `git branch --delete-merged` inched closer to merge with a concrete design for stacked-branch safety, and Tian Yuchen’s commit-reach optimization series landed with a 500× speedup for asymmetric histories. A quiet but notable milestone: Junio’s “What’s cooking” report declared the tree feature-frozen for Git 2.55, shifting the focus to regression fixes and polish.

---

## Notable threads

### Cascading indentation for `git log --graph` reaches technical completion
Pablo Sabater’s six-iteration series implementing cascading indentation for visual roots in `git log --graph` resolved its last architectural blocker today. The v6 patches introduce a clean abstraction layer in `revision.c` that insulates the graph renderer from the walker’s internal data structures, directly addressing Jeff King’s planned removal of `revs->commits`. Junio’s review conditionally approved the abstraction (patch 2/3) with a request for a defensive `BUG()` check in `revision_has_more_commits()` to guard against future misuse—particularly with `-L` line-level filtering. The core visualization logic (patch 3/3) remains unchanged from v5 and is now protected by the abstraction, with comprehensive test coverage for edge cases like filtered parents and boundary commits. The series is technically complete; the follow-up patch for defensive checks is the final hurdle before Junio’s pickup.

---

### Stacked-branch safety design finalized for `git branch --delete-merged`
Harald Nordgren’s 16-iteration series for `git branch --delete-merged` took a decisive step toward merge readiness with a concrete design for stacked-branch safety. The discussion today centered on the retargeting approach: when deleting a branch used as an upstream for another (e.g., `feature1` if `feature2` stacks on it), the proposal is to automatically update the dependent branch’s upstream to the branch that merged the deleted one (e.g., `feature2@{u} → origin/master`). Harald identified two edge cases—recursion (chains like `b1→b2→b3`) and self-referential upstreams (e.g., `b3→b3`)—and suggested handling them with warnings rather than errors, unsetting the upstream for self-references. The tone leans pragmatic: retargeting is preferable to aborting, but edge cases shouldn’t block progress. Junio’s earlier preference for abort-and-notify remains unresolved, but the design is now concrete enough for implementation in v17. The series is otherwise feature-complete, with all safety mechanisms (checked-out branches, push divergence, per-branch opt-out, `--dry-run`) implemented and tested.

---

### Shell completion aligns with shell conventions for dotfiles
Zakariyah Ali’s v3 patch series for shell completion behavior now hides dotfiles (e.g., `.gitignore`) by default, aligning Git’s path completion with standard shell conventions. The series splits the change into two patches: the first implements the behavior for `git rm`, `git mv`, and `git ls-files`, while the second extends it to all path completion contexts by making dotfile hiding the default in the `__git_index_files` helper. Junio’s feedback on code duplication and consistency has been fully addressed, and the series is now focused on final review. The change preserves explicit dotfile access (e.g., `git add .` still completes to `.gitignore`) and removes long-standing TODO comments from 2013. The implementation is clean, with AWK processing filtering dotfiles in a single place after path dequoting, and the test suite verifies both default hiding and explicit dot completion. The series is ready for merging pending maintainer sign-off.

---

### Commit-reach optimization series lands with 500× speedup
Tian Yuchen’s six-patch series optimizing `paint_down_to_common()` for one-sided histories landed today, introducing an early-exit condition that terminates the merge-base walk when one side’s queue is exhausted. The optimization targets a known bottleneck in repositories with asymmetric histories (e.g., import grafts or shallow histories), where the old implementation would traverse the entire remaining side even after the other side was exhausted. Benchmarks show 100–1000× speedups for large one-sided histories (2.6M commits), with no regressions on smaller repositories like linux.git. The series is well-structured: patches 1–2 refactor the code to introduce a `paint_queue` struct with per-side counters, patch 3 implements the core optimization (halt when one side is exhausted, provided the walk has entered the finite-generation region), and patches 4–6 add test coverage and documentation. Elijah Newren contributed test cases from his parallel work (PR #2150), and the series incorporates feedback from Derrick Stolee on the halt condition. The implementation is narrowly scoped to `commit-reach.c` and introduces no on-disk format changes. The series is ready for review and likely to graduate to `next` quickly, given the dramatic performance improvement and thorough test coverage.

---

### `git repo info` path-formatting series ready for merge after reordering
A GSoC participant’s series introducing standardized path formatting for `git repo info` cleared its final process hurdle today. Junio requested a v7 reordering to demonstrate the new `append_formatted_path()` helper replacing existing logic before adding new features, and the author confirmed the plan to squash the first two patches to show the helper immediately replacing the duplicated logic in `rev-parse`. The series adds four new fields to `git repo info` (`path.commondir.absolute`/`relative` and `path.gitdir.absolute`/`relative`) and eliminates ~80 lines of redundant code while preserving exact backward compatibility. Phillip Wood and Justin Tobler provided substantive reviews, and the implementation is technically complete. The v7 reordering is purely mechanical and won’t affect the final outcome; the series is now ready for Junio’s pickup and graduation to `next`.

---

## In brief

**Worktree diff performance investigation** -- D. Ben Knoble traced a CI hang in `t5551`/`t5559` to an HTTP/2 flow-control stall during an `ls-refs` advertisement of 100,000 loose refs on GitHub’s macOS runners. The proposed fix packs the refs (reducing advertisement time from 1.2 s to 24 ms) and aligns `$JOBS` with the runner’s actual core count, eliminating the 360-minute timeout.

**Meson `nanosec` option** -- D. Ben Knoble added a `nanosec` Meson build option to mirror Autotools’ `USE_NSEC`, enabling Git to track sub-second file timestamps. The patch is a parity fix for build system consistency, though the commit message doesn’t explain the use case for sub-second precision.

**Translation catalog output cleanup** -- A v2 patch for gitk and git-gui removes the `msgfmt --statistics` output and “Generating catalog” messages entirely, following Johannes Sixt’s suggestion to simplify by eliminating the output rather than conditionally suppressing it. The change aligns with core Git’s precedent and is ready for maintainer sign-off.

**`SubmittingPatches` design review expectations** -- Junio’s v4 patch formalizing design justification requirements for patch resubmissions landed with minor phrasing tweaks. The update instructs contributors to defend design choices on the list before revising implementations and to document how design concerns were resolved in commit messages. The patch is uncontroversial and ready for integration.

**`protect_hfs`/`protect_ntfs` follow-up** -- Tian Yuchen’s immediate follow-up patch replaced the `repo->gitdir` guard in the new accessor functions with `repo->initialized`, aligning with existing patterns for repository-scoped config fields. The change is defensive and uncontroversial, and Junio queued it directly on top of the original topic in `next`.

---

## On the radar

**Worktree diff timestamp precision** -- D. Ben Knoble’s investigation into `git diff` slowdowns in secondary worktrees identified a conditional guard in `builtin/diff.c` that may prevent refreshed index writeback. The thread is exploring runtime-configurable timestamp precision as a long-term solution, with `USE_NSEC` (nanosecond precision) as a temporary workaround. The discussion is active but not yet actionable.