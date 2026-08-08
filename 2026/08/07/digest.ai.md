# The Git Project Daily Digest – 2026/08/07

The day in brief: A **heavy traffic day** (171 emails) with **three major series reaching critical milestones**—Harald Nordgren’s `git branch --delete-merged` graduates to `next`, Tian Yuchen’s `paint_down_to_common()` optimization clears its final documentation hurdle, and Pablo Sabater’s `git cat-file --batch-command` object-type support nears the finish line. **Security concerns** surface in two threads: Junio flags a command-injection risk in a `gitk` patch, and Phillip Wood’s procedural pushback on `git history squash` stalls a long-running series. **Performance and correctness fixes** dominate the in-brief section, while **design discussions** about `git fetch` refspecs and interactive patch selection keep the radar warm.

---

## Notable threads

### `git branch --delete-merged` graduates to `next`
**Thread**: [Final v25 of Harald Nordgren's `git branch --delete-merged` series](2026/05/01/21-35-37)

Harald Nordgren’s **eight-year, 25-iteration effort** to add safe automated branch cleanup has reached a major milestone: the series is now **merged to `next`** and ready for `master`. The command (`git branch --delete-merged`) provides **order-independent stacked-branch protection**, per-branch opt-out via `branch.<name>.deleteMerged=false`, `--dry-run` preview, and the foundational `--forked` filter. Phillip Wood’s final confirmation email (2026/08/07/13-09-14) resolves the last procedural blocker, noting that the only change since v24 is a mechanical test-line wrap in patch 1/7. The series is **implementation-complete**, with all planned features, safety mechanisms, and correctness fixes in place. **Key files**: `builtin/branch.c`, `ref-filter.c`, `Documentation/git-branch.adoc`, `t/t3200-branch.sh`.

---

### `paint_down_to_common()` optimization clears final hurdle
**Thread**: [Optimize `paint_down_to_common()` for one-sided histories](2026/06/08/18-37-18)

Tian Yuchen and Kristofer Karlsson’s **100-1000x speedup** for asymmetric merge-base queries is now **technically complete** after clearing its final documentation hurdle. The series, queued in Junio’s tree, optimizes `paint_down_to_common()` to terminate early when one side of a history is exhausted, eliminating redundant traversals in repositories with import grafts or shallow histories. Elijah Newren’s latest review (2026/08/07/12-40-23) confirms the documentation is now aligned with the code, adopting "ordered" and "unordered" region terminology to clarify the semantic boundary between generation-number regions. The series touches `commit-reach.c`, `commit-graph.c`, and adds a new technical document (`paint-down-to-common.adoc`). **Performance impact**: step counts drop from 81 to 9 in the `merge-base --all commit-9-9 commit-9-1` test.

---

### `git cat-file --batch-command` adds object-type support
**Thread**: [Extend `git cat-file --batch-command` with `%(objecttype)`](2026/07/26/08-32-54)

Pablo Sabater’s GSoC series adding **object-type metadata** to `git cat-file --batch-command`’s remote-object-info queries is now **feature-complete and merge-ready** after addressing a static-analysis warning in v6. The series extends the recently merged `remote-object-info` protocol to support `%(objecttype)`, the last piece of metadata guaranteed to be identical on client and server after a fetch. Jeff King’s (Peff) refactoring (2026/08/07/22-07-01) clarifies ownership semantics in the transport layer, and the final patch (2026/08/07/22-07-05) unifies the default format for all `cat-file` batch commands. **Key files**: `fetch-object-info.c`, `protocol-caps.c`, `serve.c`, `builtin/cat-file.c`, `Documentation/git-cat-file.adoc`. The series is **backward-compatible** and has no external dependencies.

---

### `git history squash` stalls on procedural concerns
**Thread**: [v13 of `git history squash`](2026/06/14/19-25-39)

Harald Nordgren’s **eight-patch series** implementing `git history squash`—a faster, non-interactive alternative to `git rebase --autosquash`—has **stalled** after Phillip Wood’s procedural pushback (2026/08/07/13-47-53). Phillip demands **explicit replies to open questions** (including a ref-filter display bug in patch 3.5), **better documentation of behavioral changes**, and **slower iterations** to avoid fragmenting discussion. Harald’s reply (2026/08/07/18-31-04) concedes the need for a deliberate pace but **rejects Phillip’s formatting proposal**, citing Junio’s prior skepticism. The series is **technically complete** (v13 addresses all prior feedback, including Phillip’s fixup patches), but the **procedural impasse** remains unresolved. **Key files**: `builtin/history.c`, `sequencer.c`, `t/t3455-history-squash.sh`.

---

### Security concerns in `gitk` custom commands
**Thread**: [`gitk`: add user-configurable custom commands](2026/08/04/21-43-59)

Junio C Hamano’s **security review** (2026/08/07/22-16-09) flags a **command-injection risk** in Tim Wiederhake’s `gitk` patch adding custom commands to context menus. The issue: unquoted placeholder substitution (e.g., `%t` for commit title) in shell commands enables arbitrary code execution via malicious commit titles (e.g., `title?'; echo no'`). Junio’s exploit example (`echo 'title?'; echo no''`) demonstrates how an attacker could inject additional commands. The patch is **blocked** until the author addresses the issue, either by **properly quoting interpolated values**, avoiding shell execution, or restricting placeholders to safe subsets. **Key file**: `gitk-git/gitk`.

---

## In brief

- **`git rebase --update-refs` symref bugfix**: Son Luong Ngoc’s v4 series (2026/05/28/05-41-59) addresses edge cases where symbolic references cause rebase failures. Phillip Wood’s latest review (2026/08/07/15-22-33) questions the redundancy in `prepare_checked_out_branches()` and the deduplication logic for non-branch symrefs, but the series is otherwise ready for integration.

- **`git replay --linearize` UX feedback**: Justin Tobler’s review (2026/08/07/17-34-42) of Toon Claes’s v8 series questions the **single-branch restriction** for `--linearize`, suggesting the documentation could simplify its rationale (e.g., "multiple branches are not supported"). The restriction itself is uncontested, but the **terminology inconsistency** ("revision ranges" vs. "branches") may prompt minor tweaks.

- **ODB abstraction: pluggable packfile writes**: Patrick Steinhardt’s v1 series (2026/08/07/10-45-06) makes `git receive-pack` backend-agnostic by introducing `odb_transaction_write_pack()`. Junio flags a **merge conflict** with the `--no-ref-delta` topic in `seen`; the series will need a rebase before integration. **Key files**: `odb.c`, `odb.h`, `upload-pack.c`, `send-pack.c`, `builtin/bundle.c`.

- **`git fetch` refspec-mapping design**: Junio’s proposal (2026/08/07/02-42-43) to extend `--refmap` with a `matching`-like mode gains traction. The idea: allow `git fetch` to automatically update remote-tracking refs for explicitly fetched branches while preserving restricted default fetch sets. Phillip Wood’s alternative (2026/08/07/13-08-22) suggests a new `remote.<remote>.fetchMap` configuration, but Junio prefers building on existing mechanisms.

- **Interactive patch selection workflow**: Junio’s feature request (2026/08/07/04-02-43) for unified triage actions (stage/discard/defer) in `git add -p` sparks design discussion. D. Ben Knoble highlights Fugitive’s unified command set as a reference, while Christian Couder suggests leveraging `git history` for advanced use cases (e.g., squashing hunks into older commits). The thread converges on **extending `git add -p` with a "discard" action** as the immediate priority.

- **`git maintenance` flake fix**: Patrick Steinhardt’s two-patch series (2026/08/07/10-59-00) fixes a race condition in `t/t7900-maintenance.sh` by isolating auto-detach logic to specific tests. The fix is **minimal and targeted**, addressing a recurring CI pain point.

- **`USE_NSEC` runtime config**: D. Ben Knoble’s series (2026/08/07/11-56-23) converts the build-time `USE_NSEC` macro to a runtime `core.useNanosec` option. Junio’s review (2026/08/07/21-17-39) identifies a **performance-critical flaw**: the implementation calls `repo_config_get_bool()` on every `is_racy_stat()` invocation, creating a hot-path regression. A v2 iteration will cache the value in `repo_settings`.

- **Sparse index use-after-free**: Shlok Kulshreshtha’s patch (2026/08/07/19-59-40) fixes a use-after-free in `get_oid_with_context_1()` when resolving relative paths against a sparse index. The fix is **minimal and well-tested**, with a regression test in `t1092-sparse-checkout-compatibility.sh`.

---

## On the radar

- **`git history squash` procedural impasse**: Harald Nordgren’s series remains stalled until he addresses Phillip Wood’s demands for **explicit replies to open questions** and **better documentation**. The ball is in Harald’s court.

- **`git fetch` refspec-mapping design**: Junio’s `--refmap` proposal and Phillip Wood’s `remote.<remote>.fetchMap` alternative need further discussion to resolve the design direction.

- **Interactive patch selection workflow**: The thread’s focus on **extending `git add -p` with a "discard" action** may lead to a patch series, but no concrete implementation has been posted yet.

- **`gitk` custom commands security fix**: Tim Wiederhake’s patch is blocked until the **command-injection risk** is addressed. The author will need to **quote interpolated values** or avoid shell execution entirely.