# The Git Project -- Weekly Digest for 2026/07/06 -- 2026/07/12

## The period in brief

This week (2026/07/06--2026/07/12) saw **721 emails across 146 threads**, with traffic **heavier than average** and a **notably collaborative tone**. The standout developments: **`git replay --linearize` and `git history squash` graduated to `next`**, **Patrick Steinhardt’s reftable security-hardening series landed**, and **ODB abstraction took a major step forward** with `git receive-pack` switching to ODB transactions. **Performance fixes** (reftable tombstones, `unpack-trees`, commit-graph) and **usability improvements** (`git branch --delete-merged`, `git log --graph` indentation) dominated the week, while **security discussions** about lazy-fetching controls and SSH signature inspection highlighted ongoing tensions between usability and threat modeling. The three things a reader absolutely should not miss: the **reftable security hardening**, the **ODB transaction switch in `git receive-pack`**, and the **`git branch --delete-merged` series**, which offers safe automated branch cleanup with stacked-branch protection.

---

## Key developments

### Reftable backend security hardening lands
Patrick Steinhardt’s **12-patch series hardening the reftable backend against maliciously corrupted files** cleared final review and is now **queued in `next`**. The series, driven by fuzzing, addresses **out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`** in the reftable parser. Each fix is accompanied by a unit test, and the fuzzing infrastructure is now integrated into Git’s CI. The series is a **critical security effort**, ensuring the reftable backend (used in large-scale deployments) is robust against corrupted tables. No user-visible behavior changes; the hardening is purely defensive.

---

### ODB abstraction: `git receive-pack` switches to ODB transactions
Justin Tobler’s **11-patch series converting `git receive-pack` to use ODB transactions instead of `tmp_objdir`** is now **queued in `next`**, marking a major milestone in the ODB abstraction effort. The series replaces the quarantine flow with transactions, preserving existing behavior while enabling future backend-agnostic storage. The change is **internal and backward-compatible**, with no user-visible impact. Patrick Steinhardt reviewed all patches, and Junio C Hamano resolved the last design question about explicit `odb_transaction_abort()` calls in error paths. This is a **foundational step toward pluggable object databases**, enabling Git to support alternative storage backends (e.g., cloud-native, embedded) without rewriting core commands.

---

### `git branch --delete-merged` series ready for merging
Harald Nordgren’s **7-patch series introducing `git branch --delete-merged`** is **implementation-complete and ready for merging**. The command enables safe automated local-branch cleanup with **stacked-branch protection (abort-and-clear)**, per-branch opt-out via `branch.<name>.deleteMerged=false`, `--dry-run` preview, and the `--forked` filter. The v18 iteration addresses all prior feedback, including test improvements and mechanical flags cleanup. The stacked-branch protection now keeps only branches needed by unmerged branches and clears stale upstream configs for kept (merged) branches, avoiding broken dependency chains. This addresses a **long-standing workflow pain point** with careful attention to edge cases (e.g., chains of dependencies, self-referential upstreams).

---

### `git replay --linearize` graduates to `next`
Toon Claes’s **3-patch series adding `--linearize` to `git replay`** is now **queued in `next`**, offering a simpler alternative to `git rebase --no-rebase-merges` for flattening merge commits. The feature produces a linear history that avoids dangling commits and ensures a single topology even when multiple branches are involved. The series diverges from `git rebase --no-rebase-merges` by dropping merges and keeping only one branch, a design choice explicitly justified in the commit message. The only unresolved edge case—combining `--linearize` and `--revert` on a partial merge range—remains a theoretical concern but is no longer blocking. Elijah Newren raised a **usability concern** about multi-branch replay (e.g., `git replay --linearize --onto main topic1 topic2`), where the current implementation flattens all commits into a single linear history in an opaque order. The maintainers must decide whether to disallow multiple positive refs with `--linearize` or track a `last_commit` per branch to preserve branch independence.

---

### `git history squash` ready for integration
Harald Nordgren’s **5-patch series introducing `git history squash`** is **functionally complete and ready for integration**. The command folds a commit range into its oldest commit while preserving descendant history, offering a conflict-free alternative to `git rebase -i` for collapsing ranges. The v8 iteration aligns the `--reedit-message` template with `git rebase -i --autosquash`, addresses all prior feedback, and includes comprehensive test coverage. The only remaining discussion point is whether `--reedit-message` (or `--edit`) should be the default, but this is a minor usability tweak that does not block merging. The series is **marked "Will replace" by Junio**, signaling intent to merge for the next release.

---

### Reftable tombstone performance fix lands
Kristofer Karlsson’s **2-patch series fixing a quadratic performance regression in the reftable backend** is now **queued in `next`**. The regression occurred when re-creating refs after deletion (tombstoning), where the merged iterator’s `suppress_deletions` flag forced a full scan of all tombstone records before bounds checks could terminate. The fix moves tombstone suppression logic to call sites, enabling early termination. Performance improves from **~14s to ~0.2s** in the perf test’s 8000-ref scenarios. The v3 iteration addressed Patrick Steinhardt’s feedback by making `suppress_deletions` configurable via `struct reftable_stack_options`, preserving libgit2’s current behavior while future-proofing the API. This resolves a **real-world performance issue** observed by brian m. carlson in production-scale repositories.

---

### `paint_down_to_common()` optimization ready for `seen`
Tian Yuchen and Kristofer Karlsson’s **6-patch series optimizing `paint_down_to_common()`** is **ready for `seen`**, delivering **100-1000x speedups** for asymmetric merge-base queries. The series introduces early termination when one side’s commit queue is exhausted, removes the commit-date fallback (introduced in 2018), and adds Trace2 instrumentation to measure performance improvements. The series is **fully reviewed, regression-free, and self-verifying**, with exhaustive edge-case coverage. Junio C Hamano confirmed the series applies cleanly to a synthetic base, resolving the last merge conflict. This is a **major performance win** for large repositories with complex histories.

---

### `git log --graph` cascading indentation: usability decision pending
Pablo Sabater’s **4-patch series implementing cascading indentation for visual roots in `git log --graph`** is **technically complete** but stalled on a **usability decision**. The series introduces a lookahead buffer to disambiguate excluded parents, addressing all known edge cases (including `--author`/`-grep` filters). Mirko Faina proposed a **bounded zig-zag counter-proposal** to cap indentation at two levels before resetting to the left margin, avoiding excessive indentation in simple cases. Pablo countered with a **four-level wrap** to eliminate symmetry-induced ambiguity. Both designs are opt-in (triggered by `--oneline` or `--format=reference`) and require no changes to the approved lookahead buffer infrastructure. The maintainers must now rule on the indentation style and whether to retain the `--no-graph-indent` opt-out flag. The series is otherwise **ready for `next`**.

---

## In brief

**`git refs` subcommands merged** -- Patrick Steinhardt’s **5-patch series** adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` is **fully merged**, consolidating reference manipulation under a unified interface. The subcommands mirror `git update-ref` with additional usability features (e.g., `--message`, `--no-deref`).

**Rebase dropped-commit notes overhaul** -- Phillip Wood’s **11-patch systemic overhaul** fixing notes leaks in the sequencer is **queued for merging**. The series prevents notes from being copied from dropped commits during rebase, addressing a long-standing correctness issue.

**`GIT_TEST_LONG` tests reliable and CI-enabled** -- Patrick Steinhardt’s **9-patch series** makes `GIT_TEST_LONG` tests reliable and efficient enough for CI, addressing hangs and inefficiencies in GitHub and GitLab CI. The series enables `GIT_TEST_LONG` for GitLab CI pushes to integration branches (excluding Windows runners) and removes macOS RAM-disk workarounds.

**`git_hash_*()` API refactoring lands** -- Jeff King’s **7-patch series refactoring the `git_hash_*()` API** landed in `next`, making `git_hash_discard()` idempotent, standardizing wrapper usage via Coccinelle, and adding defensive runtime checks. The changes harden the API against common footguns like double-finalization or premature discarding.

**`unpack-trees` quadratic fix merged** -- Henrique Ferreiro’s **performance fix for `unpack-trees.c`** merged to `master`, resolving a quadratic-time index scan triggered by `git diff` with pathspecs matching early index entries. Performance improves from **8 minutes to 0.07 seconds** on Chromium-scale repositories.

**`git history reword` feedback requirements** -- The discussion around Pablo Sabater’s `git history reword` series clarified that feedback must cover all updated refs (not just HEAD) and that strict mode should default to on but be disableable. The command intentionally allows rewriting unreachable commits, so safety checks should not break existing workflows.

**`repo_config_values` migration advances** -- Tian Yuchen’s **9-patch series migrating global configuration variables into `struct repo_config_values`** is **under review**, with all memory leaks fixed. The series aligns the signatures of `_clear()` and `_init()` and is CI-clean.

**`git-subtree` safeguard** -- Ian Jackson’s **2-patch series** adding a safeguard to `git-subtree` to detect and reject repositories modified by a forthcoming Rust rewrite is **fully reviewed and ready for `next`**.

**`git add -e` refactoring** -- Gatla Vishweshwar Reddy’s patch refactoring `git add -e` to use Git’s internal `apply_all_patches()` API instead of forking a subprocess is **ready for merging**, with all feedback addressed.

**`USE_NSEC` deprecation** -- The discussion around `USE_NSEC` evolved into a consensus to **always compile nanosecond support** when the platform provides the necessary APIs and expose the feature via a runtime config knob (`core.useNsec`). The `USE_NSEC` build knob will be deprecated and eventually removed.

**`git merge --abort` autostash preservation** -- Kris Point’s bugfix for `git merge --abort` is **blocked pending redesign**. The current patch unconditionally saves the autostash even if it applies cleanly, violating intended behavior. An alternative approach (decoupling reset logic from `cmd_reset()`) is under discussion.

**HTTP daemon test helpers** -- Michael Montalbo’s **3-patch series fixing race conditions in HTTP daemon test helpers** is under review, replacing non-atomic shell operations with atomic `mv` and `mkdir`.

**Cygwin IPv6 regression fix** -- Ramsay Jones’s patch fixing a five-year-old regression in IPv6-style SCP URL handling on Cygwin is **blocked on a platform-compatibility concern** about Git for Windows’ UNC path support.

**`GIT_NO_LAZY_FETCH=fromAccepted`** -- Christian Couder’s series extending `GIT_NO_LAZY_FETCH` to support a `fromAccepted` mode faces a **security objection** from brian m. carlson, who argues that no additional lazy-fetching flexibility should be introduced while Git’s network code remains in C. The objection remains unresolved.

**Documentation updates** -- Junio C Hamano’s **6-patch series clarifying contributor guidance** is under review, with editorial tweaks addressing feedback. Key additions include commit-message structure guidance and `Reviewed-by:`/`Acked-by:` trailer carry-over rules.

**fsmonitor crash** -- Đoàn Trần Công Danh reported a `BUG_vfl` crash in `fsm-listen-linux.c` when Git is built without Rust (`NO_RUST=Yes`). The crash is reproducible on `next`, `seen`, and the upcoming Git 2.55.0 release. No fix yet.

---

## Looking ahead

The next week is likely to see **integration of several major series** (`git branch --delete-merged`, `git history squash`, `git replay --linearize`, reftable security hardening, ODB transactions in `git receive-pack`). **Usability decisions** for `git log --graph` indentation and `git history squash`’s default behavior will shape the final form of these features. **Security discussions** about lazy-fetching controls and SSH signature inspection may see further debate, particularly if new patches or counterarguments emerge. **Performance work** (e.g., `paint_down_to_common()` optimization) and **refactoring efforts** (e.g., `repo_config_values` migration) will continue to advance, while **platform-compatibility issues** (Cygwin IPv6, fsmonitor crash) may see fixes. The **ODB abstraction effort** is expected to progress with follow-up work on object filters and pluggable housekeeping.