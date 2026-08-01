## The day in brief.

Friday, July 31, 2026 brought a moderate but focused day on the Git mailing list—44 emails across 16 threads. The standout developments: Junio Hamano approved Kristoffer Haugsbakk’s `git-refs` documentation cleanup, Derrick Stolee’s sparse-index segfault fix advanced to `next`, and a lively debate erupted over brian m. carlson’s Git 3.0 proposal to enforce lowercase-only hex object IDs. A new `git rebase` regression with submodules also surfaced, bisected to a recent commit-graph change.

---

### Notable threads

### `git-refs` documentation cleanup lands

Kristoffer Haugsbakk’s two-patch series elevating a ref-migration warning into a prominent admonition block and adding a cross-reference to `git-maintenance` received maintainer approval. Junio Hamano’s terse “Looks good. Thanks.” signals the series is now queued for `seen` and will graduate to `next` in the next “What’s cooking” cycle. The changes are purely presentational—no code or behavior altered—but improve usability by making critical limitations harder to overlook.

---

### Sparse-index segfault fix advances to `next`

Derrick Stolee’s minimal fix for a segfault when collapsing a full index to a sparse index in the presence of an intent-to-add entry outside the sparse-checkout cone is now confirmed for the `next` branch. The patch adds a guard clause in `convert_to_sparse_rec()` to skip cache-tree nodes with negative `entry_count`, preventing an out-of-bounds access. Two new tests in `t3705` and `t1092` verify the fix and document the performance trade-off: some sparse directories remain expanded due to the ITA entry. The topic is now cooking in `next` and expected to graduate unless integration issues surface.

---

### Git 3.0 hex object ID case-sensitivity debate intensifies

brian m. carlson’s RFC series proposing a Git 3.0 breaking change to enforce lowercase-only hex object IDs continues to draw maintainer scrutiny. Junio Hamano expanded his earlier Robustness Principle objection with a concrete example: Git’s dumb HTTP transport encountering a loose object directory named in uppercase (e.g., `objects/AB/`). He argues that a robust implementation should accept and normalize such inputs rather than reject them outright, suggesting a potential compromise that preserves case-insensitive parsing for external inputs while enforcing lowercase internally. The discussion remains focused on design philosophy rather than implementation details, with no maintainer decision yet. The first five infrastructure patches are uncontroversial, but the sixth (enforcement) hinges on resolving the broader justification debate.

---

### `git rebase` regression with submodules bisected

Florian Schmidt reported a regression in `git rebase` when commit graphs are enabled and the rebase involves submodule pointer changes. The error—`fatal: invalid commit position. commit-graph is likely corrupt`—was bisected to commit `bb5da75d61` (“commit: use commit graph in `lookup_commit_reference_gently()`”), which introduced commit-graph usage in the function. The issue appears to stem from `lookup_commit_reference_gently()` resolving a submodule OID against the main repository’s commit graph, treating it as if it belonged to the main repo. Disabling commit graphs (`core.commitGraph=false`) works around the issue, but the root cause remains under investigation. The bisect result gives reviewers a clear starting point, and the report is well-structured with reproduction steps.

---

### `git add --resolved` v3 posted

Junio Hamano posted the third iteration of his `git add --resolved` feature, a workflow improvement for maintainers resolving conflicts while carrying unrelated local changes. The series consolidates duplicate conflict-marker detection logic, adds helpers for index removal with flags, and includes a performance tweak for binary files. The v3 update addresses a test-script portability issue (replacing `git checkout master` with `git checkout @{-1}`) but leaves the conceptual debate unresolved: Junio’s monolithic `--resolved` option versus Michael Montalbo’s proposed `--unmerged` selector with policy flags. The series is technically complete and ready for review, with the interface design as the primary open question.

---

### In brief

### `git config --global` documentation vs. behavior discrepancy

Nils Fahldieck and Ben Knoble reinforced the argument for aligning `git config --global`’s behavior with its documentation by making it read both `~/.gitconfig` and the XDG config file, matching `--get`’s behavior. The maintainer’s preference for documenting the single-file behavior remains unchallenged, but the cross-command inconsistency (`--get` vs. `--global`) and unintuitive behavior with `--global --no-includes` strengthen the case for a code change. Nils signaled willingness to contribute a patch but seeks guidance on updating documentation (man pages, git-scm.com, and translations).

### `git stash push` with paths stashes unrelated files

Yuri reported that `git stash push` with explicit paths unexpectedly stashes changes from an unrelated directory, including an untracked file not listed on the command line. The report is reproducible in the FreeBSD ports tree and suggests a bug in `git stash`’s path-filtering logic, possibly involving untracked files or symlinks. No patch or fix has been proposed yet.

### GSoC `git cat-file --batch-command` `%(objecttype)` support nears completion

Pablo Sabater’s Google Summer of Code series extending `git cat-file --batch-command` to support the `%(objecttype)` placeholder in remote-object-info queries is now complete at v2. The six-patch series addresses all review feedback, including patch reordering, test portability fixes, and documentation updates. Junio Hamano’s review of the first patch confirmed the dynamic request logic is safe, and the series is now ready for maintainer consideration. The implementation is narrowly scoped to object type, the last piece of metadata guaranteed to match between client and server after a fetch.

### `git add --resolved` refactoring patches

Junio’s four-patch series for `git add --resolved` includes two preliminary refactorings: consolidating duplicate conflict-marker detection logic into a shared helper and adding a new `remove_file_from_index_with_flags()` helper for dry-run and verbose modes. The patches are mechanical and uncontroversial, addressing code hygiene and paving the way for the user-facing `--resolved` option in the final patch.

### CI workflow cancellation for pull requests

Harald Nordgren proposed a GitHub Actions workflow improvement to cancel stale in-progress runs for pull requests, reducing wasted CI runner capacity. The patch modifies `.github/workflows/main.yml` to group workflow runs by pull request number and cancel in-progress runs on new pushes. The change is self-contained and unlikely to be controversial.

### Documentation standardization for `git replay` configuration

Kristoffer Haugsbakk’s `kh/doc-replay-config` series, synchronizing `replay.refAction` documentation between `git-config(1)` and `git-replay(1)`, is now fully reviewed and cooking in `next`. Toon Claes’s final visual verification on git-scm.com closed the thread, and Junio’s administrative email confirmed the v4 update is officially marked for `next`. The series touches only documentation files, with no code or behavior changes.

---

### On the radar

### `hn/history-squash` ejected from `next` pending fixups

Phillip Wood’s review exposed fundamental correctness flaws in `hn/history-squash`, leading to its ejection from `next`. The topic now resides in `seen` pending fixups for reachability logic edge cases (`UNINTERESTING` commits, root commits, multi-tip histories). The message-handling debate—whether to remove `--reedit-message`, replace it with `--edit`/`--no-edit`, or preserve `amend!`/`squash!` messages—remains unresolved. Junio’s proposed moratorium on new topics in `seen` without substantive review underscores the systemic review quality challenges highlighted by this episode.

### Trace2 recursion crash fix awaiting revised patches

Derrick Stolee’s fix for a “recursion detected in die handler” crash in the trace2 subsystem is under review, awaiting revised patches. The two-part solution—replacing `xsnprintf()` with `snprintf()` in trace2 and implementing a MinGW-specific fix for `xsnprintf()`/`libintl` redirection—remains the agreed path forward. The root cause is now confirmed as memory allocation failures in Git for Windows’s `libintl_vsnprintf()` redirection. No test coverage is expected due to the difficulty of reproducing allocation failures.