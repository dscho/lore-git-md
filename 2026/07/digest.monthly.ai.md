# The Git Project -- Monthly Digest for July 2026

**Period: 2026/07/01 -- 2026/07/31**

This month saw **2,208 emails across 543 threads**, with traffic **heavier than average** and **exceptionally eventful**. The period was defined by **major architectural progress**—ODB abstraction, reftable hardening, and `git history` subcommands all advanced significantly—while **Git 2.55.0 shipped** with parallel hooks, new built-ins, and the Linux fsmonitor daemon. The two things a reader absolutely should not miss: **`git branch --delete-merged` landed after 23 iterations**, introducing safe, automated local-branch cleanup, and **the ODB alternates performance regression sparked a fundamental architectural redesign** that will shape Git’s object storage for years. Smaller but impactful fixes—**`git mv` error messages**, **`git rebase --update-refs` symref handling**, and a **`git worktree add` memory-safety bug**—also cleared final hurdles.

---

## Key developments

### Git 2.55.0 released
**Junio C Hamano** announced the official release of Git 2.55.0, integrating **505 commits from 100 contributors**. The release is a **major milestone**, featuring:
- **Parallel hooks** (`hook.jobs`, `--jobs`), enabling concurrent execution of hooks defined in configuration.
- **New built-ins**: `git format-rev` (pretty-formatting revision expressions) and `git url-parse` (internal URL parsing logic).
- **Linux fsmonitor daemon**, complementing existing Windows/macOS backends.
- **Performance optimizations**: Faster revision traversal (priority queues for date-sorting), reachability bitmaps, and sparse-index operations in `git restore`.
- **Rust support**: Now enabled by default (opt-out), with preparatory work in `xdiff/` and the build system.
- **Breaking changes**: Stricter proxy URL validation (e.g., rejecting `httt://`) and disabled sideband terminal control sequences (except ANSI colors).

The release also includes **over 50 bugfixes**, from memory leaks to edge cases in `git bisect` and `git describe`. **Johannes Schindelin** followed up with Git for Windows 2.55.0, a downstream release bundling upstream Git 2.55.0 with Windows-specific dependencies and fixes, including a critical `git fetch` regression on older Windows 10.

**Why it matters**: This is a **feature-rich release** with significant architectural changes (ODB abstraction, Rust) and performance improvements. The volume of contributions and breadth of subsystems touched reflect active development across the project.

---

### `git branch --delete-merged` lands after 23 iterations
Harald Nordgren’s **`git branch --delete-merged`** series, in development since May, **reached technical completion and was approved for merging** after its 23rd iteration. The command introduces **safe, automated local-branch cleanup** with comprehensive safety controls: **order-independent stacked-branch protection**, **per-branch opt-out** (`branch.<name>.deleteMerged=false`), **`--dry-run` preview**, and **repeatable upstream selectors**. The v22 revision resolved the last correctness issue (order dependency in stacked-branch protection) via a **single-pass graph traversal**, and Junio C Hamano’s final approval confirmed the series is "pretty much done." The only agreed follow-up—extending stacked-branch protection to `git branch -d`—will be implemented post-merge. The series is **well-tested**, with 204 lines of new tests covering edge cases like self-referential upstreams and push divergence.

**Key participants**: Harald Nordgren, Junio C Hamano, Phillip Wood.
**Status**: **Merged to `next`**; expected in the next release.

---

### ODB alternates performance regression sparks architectural redesign
A **12% performance regression in `git receive-pack`** (server-side pushes), introduced by Patrick Steinhardt’s ODB abstraction rework, **sparked a fundamental architectural proposal**. The regression stems from a shift in object lookup order from "pack-first, loose-second" *per backend* to "pack-loose, pack-loose" *across backends*, causing redundant loose object checks in alternates directories. **Jeff King (Peff)** reframed the issue as a **systemic alternates problem**, while Patrick proposed moving alternates handling into the "files" backend itself, treating them as an implementation detail. Peff endorsed the redesign, dismissing concerns about lost flexibility as "unlikely to be practical." The proposal would enable pack-first-loose-second lookups across all alternates while simplifying commit-graph/bitmap loading. A near-term fix (targeted lookup order adjustment) may still land before Patrick’s redesign, planned for late July.

**Key participants**: Patrick Steinhardt, Jeff King, Junio C Hamano, Wolfgang Kritzinger (Atlassian).
**Status**: **Architectural proposal under discussion**; near-term fix possible.

---

### `git cat-file --batch-command remote-object-info` graduates to `next`
Pablo Sabater’s **21-patch series** enabling `git cat-file --batch-command remote-object-info` was **merged to `next`** after Junio’s explicit confirmation. The feature allows Git clients to query object metadata (size, type) from remote repositories without downloading full objects, using a new `object-info` protocol v2 capability. The v21 revision addressed the last cosmetic nits (documentation, macro formatting), and the series is **structurally unchanged** from v20, which resolved all technical concerns (dynamic capability negotiation, memory safety, server-side consistency). The implementation is **robust and well-tested**, with 747 lines of new tests covering all smart transports and edge cases.

**Key participants**: Pablo Sabater, Karthik Nayak, Junio C Hamano.
**Status**: **Merged to `next`**; on track for integration into `master`.

---

### `git history squash` lands as a rebase backend
Harald Nordgren’s **`git history squash`** series, which efficiently collapses a linear or merge-commit-containing range into its oldest commit while preserving descendant history, **reached its tenth iteration and was queued for `next`**. The command aligns with `git rebase -i`’s squash-message template and includes robust safety checks (e.g., rejecting ranges with multiple bases). Junio C Hamano **fully endorsed** the vision of using `git history squash` and `git replay` as a **faster, non-interactive backend for `git rebase --autosquash`**, calling it "an ideal future." This series is a **foundational piece** of that effort, eliminating the repeated conflict resolution of the current rebase implementation.

**Key participants**: Harald Nordgren, Junio C Hamano.
**Status**: **Merged to `next`**; expected in the next release.

---

### Reftable backend: security hardening and performance fixes
Patrick Steinhardt’s **12-patch series hardening the reftable backend against maliciously corrupted files** cleared its final review hurdle and is now **ready for merging**. The series, driven by fuzzing, addresses **out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`**, with each fix accompanied by a unit test. Performance regressions were also addressed: Kristofer Karlsson’s **two-patch series fixing a quadratic-time tombstone handling issue** (improving runtime from ~14s to ~0.2s for 8000 refs) is now **final-review complete** and ready for `next`. These changes collectively **harden the reftable backend for production use** while ensuring it scales efficiently under heavy ref churn.

**Key participants**: Patrick Steinhardt, Kristofer Karlsson, Toon Claes.
**Status**: **Ready for merging**; performance fixes merged to `next`.

---

### `git replay --linearize`’s multi-branch ambiguity remains unresolved
Toon Claes’s **seven-month effort to add a `--linearize` option to `git replay`** (flattening merge commits into a linear history) was **queued in `next`** after resolving all prior feedback. However, a **philosophical debate** between **flexibility (Junio C Hamano)** and **strictness (Elijah Newren)** has stalled progress. The current implementation flattens all specified branches into a single linear history, concatenating them in an order determined by the revision walk (not the user’s command-line order).

**Key positions**:
- **Junio**: The current behavior is **intentional and reasonable**. Users can avoid concatenation by invoking `git replay` multiple times.
- **Elijah**: The current behavior **violates the principle of least astonishment**, creates a **recoverability asymmetry**, and is an **emergent, unintended behavior**. He proposes a **minimal fix**: restrict `--linearize` to a single positive ref.

**Why it matters**: This debate **highlights a tension** between flexibility and predictability in Git’s design. The outcome will set a precedent for how future commands handle **multi-branch operations**.

**Status**: **Queued in `next` but blocked by design debate**.

---

### Memory-safety bug in `git worktree add` can cause data loss
Matthias Aßhauer posted a **critical bugfix** for `git worktree add` that prevents an out-of-bounds read leading to **silent data corruption** (recursive deletion of the working directory, including `.git`). The bug surfaces when an empty string or a string of only directory separators (e.g., `""` or `"//"`) is passed as the worktree path, likely via an unset environment variable. Junio’s review identified a **logical flaw** in the proposed fix (potential pointer underflow), and a v2 is expected soon. The series is **under active review** and high-priority.

**Key participants**: Matthias Aßhauer, Junio C Hamano.
**Status**: **Bugfix in progress**; v2 expected.

---

### `diff.<driver>.process` RFC reaches technical completion
Michael Montalbo’s **9-patch RFC** for external diff/blame tool integration **reached technical completion** after resolving its last merge conflict. The series introduces a **long-running subprocess protocol** that lets tools inject hunks into Git’s pipelines while preserving downstream features (word diff, `--stat`, `git blame`, `git log -L`). The v6 revision wired `--stat`/`--numstat` to the external process and completed the integration for `git log -L`. Junio’s final sign-off confirmed the series is unblocked; the only remaining cases using Git’s builtin diff are combined diffs (`--cc`). The protocol’s design—**forward-compatible, cache-aware, and robust to tool failures**—sets a new standard for external tool integration.

**Key participants**: Michael Montalbo, Junio C Hamano, Eric Sunshine.
**Status**: **Ready for `next`**.

---

## In brief

**`git mv` misleading error messages fixed** -- Lucas Zamboni Orioli’s **2-patch series** fixing misleading `git mv` errors (e.g., blaming the source file when the destination directory is missing) **cleared its last design hurdle**. The series improves error messages to include both source and destination paths, adds an early existence check for the destination directory, and includes **83 new test lines** covering symlink edge cases.
**Status**: **Merged to `next`**.

**`git rebase --update-refs` symref bugfix nears merge** -- Son Luong Ngoc’s **2-patch series** fixing edge cases with symbolic references in `git rebase --update-refs` **cleared its last design hurdle**. The series handles **non-branch symrefs** (e.g., tags) and **cross-worktree conflicts**, with **83 new test lines** covering symlinks as both final and intermediate components.
**Status**: **Merged to `next`**.

**`git log --graph` visual root indentation** -- Pablo Sabater’s **v12 series** introducing indentation for "visual roots" (commits with excluded parents) in `git log --graph` is **ready for `next`**. Features a lookahead buffer, four-level wrap, and opt-out mechanism.
**Status**: **Ready for `next`**.

**`the_repository` removal efforts** -- Two series advanced:
- **Tian Yuchen’s v12** (migrating global config variables into `struct repo_config_values`) is **ready for `next`**.
- **René Scharfe’s 5-patch series** (introducing repository-aware tempfile/lockfile APIs) is **queued in `seen`**.
**Status**: **Ready for `next`**.

**`git stash rename`** -- User testimonials reinforced the justification for the feature, but **Junio’s skepticism persists** about the current stash design. No progress on implementation.
**Status**: **Stalled**.

**Packfile URI race conditions** -- Ted Nyman’s v4 series fixing race conditions in Git’s HTTP transport during concurrent packfile URI and dumb HTTP downloads remains **blocked by test flakiness**. Jeff King identified a **false positive** in the `--stress` test, and the fix is to **drop the `test -s` check**.
**Status**: **Blocked by test flakiness**; v5 expected.

**`git send-email` filesystem path completion** -- Yury Norov’s patch to extend bash completion for `git send-email` to include filesystem paths remains **under review**, with a **usability trade-off unresolved**.
**Status**: **Under review**; v2 expected.

**`git repo info` path keys** -- K Jayatheerth’s **7-patch series** adding path-related keys (e.g., `path.toplevel`, `path.hooks`) to `git repo info` is now **mechanically clean and ready for substantive review**.
**Status**: **Ready for review**.

**CI toolchain upgrades** -- Jeff King’s **2-patch series** bumping the Ubuntu image for static-analysis jobs to `ubuntu-24.04` (picking up Coccinelle 1.3.0+) received **Junio’s approval**.
**Status**: **Merged to `master`**.

**`git config --url` gains scp-style URL support** -- Fabian Pottbäcker’s **3-patch series** adding scp-style SSH URL support (e.g., `host:path`) to `git config --url` is **uncontroversial and ready for review**.
**Status**: **Ready for review**.

**`git clone --revision` segfault fix** -- Adrian Friedli’s **one-line NULL check** to fix a segfault when using `--revision` with servers that advertise more refs than requested has been **merged to `master`**.
**Status**: **Merged**.

**UTF-8 string-width refactoring** -- Hardik Kumar’s **v2 patch** changing `utf8_strwidth()` to return `size_t` addressed reviewer concerns by updating all callers. The **overflow check removal** remains unresolved.
**Status**: **Under review**.

---

## Looking ahead

### Topics likely to dominate August
- **ODB alternates redesign**: Patrick Steinhardt’s **architectural shift** (moving alternates into the "files" backend) is the most consequential in-flight work, with **Peff’s endorsement** boosting its credibility. A near-term fix (targeted lookup order adjustment) may land before the redesign.
- **`git stash` redesign**: Junio’s suggestion to **tie stashes to branches** could subsume the `rename` feature, but no concrete proposals yet.
- **Packfile URI race conditions**: Ted Nyman’s series is **blocked by test flakiness**, but the fix is straightforward (drop the `test -s` check). The core logic is sound, and the series is otherwise ready for `next`.
- **`git worktree add` memory-safety bug**: Matthias Aßhauer’s bugfix is **high-priority**, with a v2 expected soon.
- **`git replay --linearize`**: Toon Claes must respond to the design debate between Junio and Elijah. The series remains in `next` but is unlikely to graduate without resolution.

### Ongoing efforts to watch
- **Rustification**: No updates; the effort remains **contentious** (Randall S. Becker’s NonStop port concerns vs. memory safety goals).
- **`paint_down_to_common()` optimization**: Elijah Newren’s **resumed review** of the technical document surfaced precision gaps around v1 commit-graph handling. Kristofer Karlsson proposed a `commit_graph_generation_topo_ceiling()` helper to clarify invariants.
- **`git maintenance` gc-promisor task**: The proposed `git maintenance run --task=gc-promisor` subcommand remains under design discussion, with performance testing on large promisor packs underway.