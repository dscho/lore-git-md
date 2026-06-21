# The Git Mailing List Daily Digest for 2026/06/20

**The day in brief.**
A Saturday with moderate traffic (37 emails in 14 threads) saw steady progress on long-running feature series and a flurry of small but meaningful cleanups. The standout developments: Pablo Sabater’s `git log --graph` indentation series cleared its last architectural hurdle, and Harald Nordgren’s `git branch --delete-merged` inched closer to final design with a concrete proposal for stacked-branch retargeting. Quieter threads delivered build-system parity, CI fixes, and a performance optimization for merge-base queries.

---

## Notable threads

### Cascading indentation for `git log --graph` reaches technical completion
Pablo Sabater’s six-iteration series implementing cascading indentation for visual roots in `git log --graph` has resolved its last architectural blocker. The v6 patches introduce a clean abstraction layer in `revision.c` that insulates the graph renderer from the walker’s internal data structures, directly addressing Jeff King’s planned removal of `revs->commits`. Junio C Hamano conditionally approved the abstraction (patch 2/3) pending a follow-up patch that adds a defensive `BUG()` check in `revision_has_more_commits()` to guard against future misuse, particularly with line-level log filtering (`-L`). The core visualization logic (patch 3/3) remains unchanged from v5 and is now protected by the abstraction. With all technical concerns resolved, the series is poised for final review once the defensive check is implemented.

---

### Stacked-branch retargeting proposed for `git branch --delete-merged`
Harald Nordgren’s 16-iteration series adding `git branch --delete-merged` took a decisive step toward final design with a concrete proposal for handling stacked branches. When a branch is used as an upstream for another unmerged branch (e.g., `feature2` stacking on `feature1`), the current implementation aborts deletion to avoid breaking `@{u}` references. Nordgren now proposes automatically retargeting the dependent branch’s upstream to the branch that merged the deleted one (e.g., `feature2@{u} → origin/master`), with warnings for edge cases like recursive chains (`b1→b2→b3`) and self-referential upstreams (`b3→b3`). The approach mirrors GitHub’s stacked-PR automation but must handle Git’s decentralized nature. Junio’s initial preference for abort-and-notify remains unresolved, but the proposal provides a clear path forward for v17.

---

### Shell completion aligns with shell conventions
Zakariyah Ali’s v3 patch series completed the effort to hide dotfiles (e.g., `.gitignore`) in Git’s path completion unless explicitly requested with a leading dot. The series now splits the change into two patches: the first implements the behavior for `git rm`, `git mv`, and `git ls-files`, while the second extends it to all path completion contexts by making dotfile hiding the default in the `__git_index_files` helper. Junio C Hamano’s feedback on code duplication and consistency has been fully addressed, and the series is now focused on final review. The change aligns Git’s completion with standard shell conventions and resolves long-standing TODO comments from 2013.

---

### Performance optimization for merge-base queries
Tian Yuchen’s six-patch series introduced an early-termination optimization for `paint_down_to_common()` in Git’s commit-reach subsystem. The optimization halts the graph walk when one side of a merge-base query exhausts its commit queue, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts or shallow histories). Benchmarks show 100–1000x speedups for asymmetric histories (2.6M commits) with no regressions on smaller repositories. The series is well-structured, with refactoring separated from behavior changes, and includes comprehensive test coverage for edge cases (e.g., criss-cross merges, grafts, shallow histories) contributed by Elijah Newren. The core optimization (patch 3/6) is authored by Kristofer Karlsson, reflecting prior feedback from Derrick Stolee and Elijah Newren.

---

## In brief

**`git repo info` gains standardized path formatting** -- A GSoC series from Lucas Seiki Oshiro introduced a shared `append_formatted_path()` helper in `path.c`, eliminating duplicated path-formatting logic in `rev-parse` and adding new fields (`path.commondir.absolute`/`relative` and `path.gitdir.absolute`/`relative`) to `git repo info`. Junio requested a v7 reordering to demonstrate the helper replacing existing logic before adding new features, but the series is otherwise complete and ready for merging.

**macOS linker warning suppression lands** -- Harald Nordgren’s patch suppressing macOS linker warnings about duplicate library references was confirmed working on older macOS systems (Xcode 14.2) with both Makefile and Meson builds. The patch adds `-Wl,-no_warn_duplicate_libraries` to `config.mak.uname`, version-gated for Xcode 15+, and is now queued in Junio’s `next` branch.

**Build system parity for sub-second timestamps** -- D. Ben Knoble added a `nanosec` option to Meson’s build configuration, mirroring the Autotools `USE_NSEC` knob. The option enables Git to track sub-second file modification and change times when enabled, ensuring feature parity between the two build systems.

**CI fix for macOS GitHub Actions runners** -- Michael Montalbo identified and fixed a CI hang in `t5551` and `t5559` caused by an HTTP/2 flow-control stall on GitHub’s virtualized macOS runners. The fix packs the 100,000 refs in the test and aligns the CI job’s parallelism with the runner’s actual core count, reducing wall-clock time from 360 minutes (timeout) to 30–44 minutes.

**Translation catalog output silenced** -- A v2 patch from an unnamed contributor removed the `msgfmt --statistics` output and "Generating catalog" messages from gitk and git-gui’s Makefiles, following Johannes Sixt’s suggestion to simplify by eliminating the output entirely rather than conditionally suppressing it. The change aligns with core Git’s precedent and is ready for maintainer review.

**`SubmittingPatches` clarifies design review expectations** -- Junio C Hamano’s v4 patch formalized the expectation that contributors must engage with design-level critiques before revising their implementation. The update softens the prescriptive tone of earlier versions while preserving the core intent, adding concrete examples of design critiques (e.g., "questioning if the feature is worth implementing"). The patch is ready for integration.

---

## On the radar

**Worktree diff performance investigation** -- D. Ben Knoble’s ongoing investigation into `git diff` slowdowns in secondary worktrees traced the issue to a conditional guard in `builtin/diff.c` that appears to prevent refreshed index writeback. The thread is exploring whether the guard is truly non-functional or serves an undocumented purpose, with `USE_NSEC` (nanosecond timestamp precision) emerging as a potential workaround. Knoble plans to add `USE_NSEC` support to the Meson build system, though the thread’s earlier discussion suggests runtime configuration might be a better long-term solution.