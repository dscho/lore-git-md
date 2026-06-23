# The Git Project Mailing List Daily Digest for 2026/06/22

## The day in brief
A busy but focused day on the Git mailing list, with **119 emails across 27 threads**. The standout developments: **Junio C Hamano merged several long-running series** including Patrick Steinhardt's GitLab CI Windows improvements and Taylor Blau's `--delta-islands`/`--path-walk` integration, while **architectural debates continued** around Toon Claes's `git replay --linearize` and Harald Nordgren's `git branch --delete-merged`. The day also saw **critical design resolutions** in Patrick Steinhardt's ref backend lazy-loading series and **performance optimizations** in Tian Yuchen's `paint_down_to_common()` work.

---

## Notable threads

### GitLab CI Windows improvements merged for Git 2.55
**Topic**: `ps/gitlab-ci-windows` (Patrick Steinhardt)
**What happened**: Junio C Hamano **merged Patrick Steinhardt's patch** migrating GitLab CI Windows builds away from Chocolatey, addressing reliability issues that had plagued CI pipelines. The change reduces macOS CI runtime from 360 minutes to 30-44 minutes and is now **confirmed for inclusion in Git 2.55**.
**Why it matters**: This resolves a long-standing CI flakiness issue that had been "extremely annoying" for developers, particularly during the Git 2.55 release cycle. The patch touches only `.gitlab-ci.yml`, making it a low-risk, high-impact improvement.
**What's next**: The change will appear in the next "What's cooking" report as merged to `master`.

---

### `git replay --linearize` at architectural crossroads
**Topic**: `[PATCH v4 0/3] replay: introduce --linearize option` (Toon Claes)
**What happened**: The series to add `--linearize` support to `git replay` (flattening merge commits like `git rebase --no-rebase-merges`) **hit a critical design debate**. Junio C Hamano and Patrick Steinhardt raised concerns about:
1. **Patch 1/3's boolean refactoring** (converting `enum replay_mode` to a `reverse` flag), arguing it obscures callsite meaning (e.g., `create_commit(..., true)`) and limits future extensibility.
2. **Patch 3/3's interface scope**, questioning whether `--linearize` should hardcode only `--no-rebase-merges` or mirror `git rebase`'s full flexibility (`--rebase-merges`, `--rebase-merges=rebase-cousins`).
**Why it matters**: The series is otherwise technically complete, with all prior feedback addressed. The boolean vs. enum debate touches on a broader tension in Git's codebase between simplicity and future-proofing.
**What's next**: Toon Claes must respond to the architectural concerns before Junio's "Will merge to 'next'" can proceed. A v5 may be needed to address the boolean refactoring.

---

### Ref backend lazy-loading series reaches completion
**Topic**: `[PATCH v5 00/11] refs: implement lazy-loading for ref backends` (Patrick Steinhardt)
**What happened**: Patrick Steinhardt **completed the v5 rewrite** of his ref backend series, implementing lazy-loading to resolve recursive initialization issues with "onbranch" configuration. The series now:
- **Defer write-config parsing** until the first write operation (e.g., `core.logAllRefUpdates`, `reftable.blockSize`).
- **Add a recursion guard** to `get_main_ref_store()` as a safeguard.
- **Unify files and reftable backends** under the same lazy-loading design.
**Why it matters**: This addresses a long-standing chicken-and-egg problem where early config reads (e.g., `includeif.onbranch`) could trigger recursive ref store access. The redesign eliminates the v4 `ignore_refs` workaround and aligns with Jeff King's advocacy for minimizing early config loading.
**What's next**: The series is ready for final review, with Karthik Nayak and Junio likely to scrutinize the reftable backend's dynamic write option handling.

---

### `git branch --delete-merged` series finalizes stacked-branch protection
**Topic**: `[PATCH v17 0/7] branch: introduce --delete-merged` (Harald Nordgren)
**What happened**: The series to add `git branch --delete-merged` (safe cleanup of merged local branches) **resolved its last major design question**: stacked-branch protection. The command now **aborts with a clear error** if a branch is used as an upstream for another branch (e.g., "Cannot delete 'feature1' because 'feature2' depends on it"), preserving the entire dependency chain. This replaces the earlier upstream-clearing approach and aligns with Junio's preference for abort-and-notify.
**Why it matters**: The series introduces a **new safety mechanism** for branch management, addressing a common pain point in topic branch workflows. The `--dry-run` flag and per-branch opt-out (`branch.<name>.deleteMerged=false`) make it flexible for users.
**What's next**: The series is now **feature-complete** and ready for final review, pending minor polish (e.g., test cases for upstream-clearing messages).

---

### `paint_down_to_common()` optimization nears consensus
**Topic**: `[PATCH v1 0/6] commit-reach: optimize paint_down_to_common() for one-sided histories` (Tian Yuchen)
**What happened**: The series to optimize `paint_down_to_common()` (early termination for one-sided histories) **resolved all high-weight objections** in v1. Key agreements:
- **Termination condition**: `(!queue.p1_count || !queue.p2_count) && !queue.pending_merge_bases` (replacing the addition-based check).
- **Finite-generation guard**: Early exit only after entering the finite-generation region.
- **Performance instrumentation**: **hyperfine** for end-to-end timing, **trace2_data_intmax()** for "commits walked" metrics.
**Why it matters**: The optimization targets a known bottleneck in repositories with import grafts or shallow histories, where asymmetric histories force full traversal of one side. The 500x+ speedup claimed in the series could significantly improve `git merge-base` performance.
**What's next**: Tian Yuchen is preparing v2 with reordered tests, performance benchmarks, and stylistic refinements.

---

### `git log --follow` improvements merged
**Topic**: `[PATCH] t4216: remove no-op test for signed char` (Miklos Vajna)
**What happened**: Junio C Hamano **merged Miklos Vajna's patch** improving `git log --follow` for non-linear history, specifically rename tracking across merge commits. The series uses a slab (`follow_pathspec_slab`) to track per-commit pathspecs, replacing the problematic global variable approach.
**Why it matters**: This removes a long-standing limitation in Git's rename tracking, where `--follow` failed to work well on non-linear histories. The solution is elegant, using existing diff machinery with per-commit state rather than expanding Git's path tracking capabilities.
**What's next**: The series is now in `master`, with no further changes expected.

---

## In brief

> **`git cat-file --batch-command` security-hardened series** -- Pablo Sabater's GSoC series (v13) implementing `remote-object-info` for querying object metadata from remotes **faces three unresolved concerns**: a memory management issue in patch 10/12 (unnecessary `static` declarations), a protocol compatibility regression risk in patch 6/12 (`GIT_HASH_SHA1_LEGACY` substitution), and two documentation nits. Karthik Nayak's substantive review flags the protocol change as potentially breaking backward compatibility with older Git versions.

> **`git history squash` series** -- Harald Nordgren's series to fold commit ranges into a single commit **awaits resolution of a conceptual question**: whether to support an `--ancestry=` flag (like `git log`) to control range interpretation. Patrick Steinhardt requested a response from the author before final review.

> **`git branch --set-upstream-to` usability improvements** -- Harald Nordgren's series to improve error messages for common misformattings (e.g., `git branch --set-upstream-to=origin main`) **faces maintainer skepticism**. Junio C Hamano argues the current error message ("branch 'main' does not exist") is accurate and suggests the advice should only appear when `origin/main` exists. The series is on hold pending clarification of the "slip" terminology for topic branch naming.

> **`ignore_case` libification merged** -- Justin Tobler and Tian Yuchen's series moving `ignore_case` into `struct repo_config_values` **was merged**, eliminating a global variable. The series aligns with the broader `the_repository` removal initiative.

> **macOS CI hang root cause identified** -- Patrick Steinhardt diagnosed the root cause of CI hangs in `t5551` and `t5559`: **Apache's default 300-second timeout** is being hit during `ls-refs` advertisement of 100,000 refs. The thread now focuses on whether to increase the timeout, optimize the test further, or extend keepalive mechanisms to cover `ls-refs`.

> **`USE_NSEC` Meson parity patch** -- D. Ben Knoble's patch to add a `nanosec` Meson option for `USE_NSEC` **sparked a debate** about whether the knob should exist at all. Jeff King and Patrick Steinhardt raised concerns about its reliability on filesystems with coarse timestamp granularity (e.g., CEPH, CIFS, NTFS, UDF). The patch remains queued for merging, but its long-term viability is in question.

---

## On the radar

> **`git replay --linearize`** -- The series is **one architectural decision away** from merging. Toon Claes must respond to Junio and Patrick's concerns about the boolean refactoring and interface scope.

> **Ref backend lazy-loading** -- Patrick Steinhardt's v5 series is **ready for final review**, with Karthik Nayak likely to focus on the reftable backend's dynamic write option handling.

> **`paint_down_to_common()` optimization** -- Tian Yuchen's v2 will include **hyperfine benchmarks** and reordered tests, addressing the last procedural feedback.

> **`git branch --delete-merged`** -- Harald Nordgren's v17 is **feature-complete** and ready for final review, with stacked-branch protection now finalized as abort-and-notify.

> **`git cat-file --batch-command`** -- Pablo Sabater's series **needs resolution of the protocol compatibility regression risk** before merging. The `GIT_HASH_SHA1_LEGACY` substitution in patch 6/12 may break backward compatibility with pre-v2 servers.