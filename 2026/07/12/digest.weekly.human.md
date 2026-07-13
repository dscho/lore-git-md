# The Git Project -- Weekly Digest for 2026/07/06 -- 2026/07/12

## The period in brief

This week (2026/07/06--2026/07/12) saw **782 emails across 166 threads**, marking a **heavy but productive** period for Git development. Traffic was **routine in volume but eventful in content**, with **multiple long-running efforts reaching critical milestones**: `git replay --linearize` and `git history squash` were queued for `next`, the reftable backend received **security hardening and performance fixes**, and the ODB abstraction effort advanced with `git receive-pack` now using ODB transactions. The standout developments: **Patrick Steinhardt’s reftable security-hardening series (12 patches) cleared final review**, **Junio queued Toon Claes’s `git replay --linearize` series after months of iteration**, and **Harald Nordgren’s `git branch --delete-merged` series (v18) became implementation-complete**. Two **late-stage usability debates**—one on `git log --graph` indentation, the other on `USE_NSEC` deprecation—dominated discussion toward the week’s end.

---

## Key developments

### Reftable backend: security hardening and performance fixes
Patrick Steinhardt’s **12-patch series hardening the reftable backend against maliciously corrupted files** cleared its final review hurdle and is now **ready for merging**. The series, driven by fuzzing, addresses **out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`**, with each fix accompanied by a unit test. Performance regressions were also addressed: Kristofer Karlsson’s **two-patch series fixing a quadratic-time tombstone handling issue** (improving runtime from ~14s to ~0.2s for 8000 refs) is now **final-review complete** and ready for `next`. These changes collectively **harden the reftable backend for production use** while ensuring it scales efficiently under heavy ref churn.

### ODB abstraction: `git receive-pack` now uses ODB transactions
Justin Tobler’s **11-patch series converting `git receive-pack` to use ODB transactions instead of `tmp_objdir`** was **queued for `next`**, marking a major step toward pluggable object databases. The series preserves existing behavior while enabling future backend-agnostic storage, and carries Patrick Steinhardt’s Reviewed-by on all patches. This is the **first user-facing command to adopt the ODB abstraction**, setting a precedent for other commands (e.g., `git fetch`, `git gc`) to follow.

### `git replay --linearize` graduates to `next`
Toon Claes’s **seven-month effort to add a `--linearize` option to `git replay`** (flattening merge commits into a linear history) was **queued in `next`** after resolving all prior feedback. The series diverges from `git rebase --no-rebase-merges` by dropping merges and keeping only one branch, a design choice explicitly justified in the commit message. A **late-stage design question**—whether to disallow multiple positive refs with `--linearize`—remains unresolved but does not block integration. The series is now **cooking in `next`** alongside other topics.

### `git history squash` and `git history reword` near completion
Harald Nordgren’s **`git history squash` series (v8)**—collapsing a commit range into its oldest commit while preserving descendant history—is **functionally complete and ready for integration**. The series aligns with `git rebase -i`’s squash-message template and includes robust safety checks (e.g., rejecting ranges with multiple bases). Pablo Sabater’s **`git history reword` series** (rewriting commit messages in a range) also advanced, with feedback clarifying that **strict mode should default to on but be disableable**. Both commands are **poised to land in the next release**, offering conflict-free alternatives to `git rebase -i`.

### `git branch --delete-merged` becomes implementation-complete
Harald Nordgren’s **`--delete-merged` series (v18)**—enabling safe automated local branch cleanup with **per-branch opt-out**, **stacked-branch protection (abort-and-clear)**, and **`--dry-run` preview**—is now **implementation-complete**. All substantive review feedback has been addressed, including test improvements and support for `<branch>` arguments. The series is **ready for merging** pending a mechanical flags cleanup in patches 2–4/7.

### `git log --graph` cascading indentation: usability debate
Pablo Sabater’s **v9 series implementing cascading indentation for visual roots in `git log --graph`** is **technically complete**, with all edge cases (including `--author`/`-grep` filters) resolved. However, a **late-stage usability debate** emerged over whether to adopt **bounded zig-zag indentation** (Mirko Faina’s proposal) to avoid excessive indentation in simple cases. The maintainers must now rule on the indentation style and whether to retain the `--no-graph-indent` opt-out flag. The lookahead buffer infrastructure (patches 2–3/4) is uncontroversial and could land independently.

### `USE_NSEC` deprecation: runtime config replaces build-time knob
The discussion around D. Ben Knoble’s patch to add a `nanosec` Meson option (mirroring Autotools’ `USE_NSEC`) evolved into a **broader rethink of sub-second timestamp handling**. Consensus now favors:
1. **Always compiling nanosecond support** when the platform provides the necessary APIs.
2. **Exposing the feature via a runtime config knob** (`core.useNsec`).
3. **Deprecating and eventually removing the `USE_NSEC` build knob** (Meson and Makefile).
Jeff King’s testing confirmed that nanosecond timestamps are preserved correctly on modern filesystems, obsoleting historical "racy Git" concerns. A new patch series implementing `core.useNsec` is expected.

---

## In brief

**`paint_down_to_common()` optimization** -- Tian Yuchen and Kristofer Karlsson’s **v6 series optimizing `paint_down_to_common()`** (delivering 100-1000x speedups for asymmetric merge-base queries) is **ready for `seen`** after clearing its last procedural hurdle. The series is **fully reviewed, regression-free, and self-verifying**.

**`repo_config_values` migration** -- Tian Yuchen’s **v10 series migrating nine global config variables into `struct repo_config_values`** is **CI-clean and memory-leak-free**, with all feedback addressed. The series is a solid step in the `the_repository` removal effort.

**`git-subtree` safeguard** -- Ian Jackson’s **patch series adding a safeguard to `git-subtree`** (detecting Rust rewrite usage) is **awaiting v2** with test style improvements. The series prevents mixed-tool corruption in `contrib/`.

**Cygwin IPv6 regression fix** -- Ramsay Jones’s **patch fixing a five-year-old regression in IPv6-style SCP URL handling on Cygwin** is **blocked on a platform-compatibility debate** with Torsten Bögershausen. The fix is technically sound but may conflict with Git for Windows’ UNC path support.

**`GIT_NO_LAZY_FETCH=fromAccepted`** -- Christian Couder’s **series extending `GIT_NO_LAZY_FETCH` to support a `fromAccepted` mode** faces a **high-weight security objection** from brian m. carlson. The objection remains unresolved, but Christian committed to improving documentation in v2.

**`git add -e` refactoring** -- Gatla Vishweshwar Reddy’s **patch replacing the subprocess call to `git apply` with the internal `apply_all_patches()` API** is **ready for merging**, with all feedback addressed in v4.

**Double-free fix** -- Patrick Steinhardt’s **bugfix for a double-free in the object-file subsystem’s stream handling** is **ready to merge**, with Jeff King confirming the fix and test.

**Documentation updates** -- Junio C Hamano’s **six-patch series clarifying contributor guidance** saw **editorial tweaks** in response to feedback, including a revised commit-message template and disambiguation of the `seen` branch’s role.

**fsmonitor crash** -- Đoàn Trần Công Danh reported a **`BUG_vfl` crash in `fsm-listen-linux.c`** when Git is built without Rust (`NO_RUST=Yes`). The crash is reproducible on `next`, `seen`, and the upcoming Git 2.55.0 release. No fix yet.

---

## Looking ahead

The next week is likely to see **integration of several queued series**, including:
- **`git replay --linearize`** (already in `next`).
- **`git history squash`** and **`git history reword`** (both functionally complete).
- **`git branch --delete-merged`** (awaiting a mechanical flags cleanup).
- **Reftable security hardening** and **performance fixes** (both final-review complete).
- **ODB transaction adoption in `git receive-pack`** (queued for `next`).

**Usability debates** will dominate discussion:
- **`git log --graph` indentation**: Maintainers must rule on the bounded zig-zag design.
- **`USE_NSEC` deprecation**: A new patch series implementing `core.useNsec` is expected.
- **`GIT_NO_LAZY_FETCH=fromAccepted`**: The security objection remains unresolved; v2 is pending.

**Performance-critical regressions** warrant prompt attention:
- **fsmonitor crash** (reproducible on `next` and `seen`).
- **Cygwin IPv6 regression** (blocked on platform-compatibility concerns).

**Long-term efforts** continue to advance:
- **ODB abstraction**: More commands (e.g., `git fetch`, `git gc`) are expected to adopt ODB transactions.
- **`the_repository` removal**: The `repo_config_values` migration series is a solid step forward.
- **Rustification**: Build system adjustments for macOS Universal Binary support landed, but broader integration remains contentious.