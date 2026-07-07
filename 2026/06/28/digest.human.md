Here’s the daily digest for **2026/06/28**, capturing the most consequential developments from the Git mailing list:

---

# The day in brief
**2026/06/28** was a **high-volume, milestone-heavy day** (45 emails, 17 threads) with **three major series landing**, a **post-merge regression surfacing**, and **critical performance optimizations** advancing. The standout threads: **Patrick Steinhardt’s reftable security hardening** merged into `next`, **Tian Yuchen’s `paint_down_to_common()` optimization** delivered 100-1000x speedups, and **Pablo Sabater’s GSoC `git cat-file --batch-command`** reached final readiness. A **regression in `git replay --linearize`** emerged late in the day, requiring urgent follow-up.

---

## Notable threads

### Reftable security hardening lands in `next`
Patrick Steinhardt’s **11-patch series** hardening Git’s reftable backend against corrupted files was **merged into `next`** after addressing all review feedback. The series fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in the reftable parser, and introduces a **libFuzzer-based fuzzer** to prevent regressions. Key patches:
- **Patch 5/11**: Fixes a heap-buffer-overflow in log block reconstruction.
- **Patch 11/11**: Fixes an OOB read on truncated tables.
- **Patches 1–2**: Add fuzzing infrastructure and CI integration.
The series is **internal-only** (no user-facing changes) and critical for the reftable backend’s stability. The fuzzer now runs for 2+ hours without finding new issues, setting a new bar for security testing in Git.

> **Why it matters**: This is the first time Git’s reftable backend has undergone systematic fuzzing, and the fixes address latent bugs that could crash Git or corrupt refs. The fuzzer will now run continuously in CI, catching regressions early.

---

### `paint_down_to_common()` optimization merged
Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged**. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Key improvements:
- **Regression fix**: Addressed an unconditional BUG assertion in patch 7/8.
- **Code clarity**: Unified halt conditions in the `paint_state` struct.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).
The optimization is a **major win for `git merge-base` performance** in asymmetric histories, with no known regressions.

> **Why it matters**: This is the first significant performance improvement to Git’s merge-base machinery in years. Users with large, asymmetric histories (e.g., monorepos with import grafts) will see **100-1000x speedups** in `git merge-base` and related commands.

---

### `git replay --linearize` regression surfaces
A **post-merge regression** in `git replay --linearize` (merged in v5) was reported by Johannes Schindelin: the command **silently drops commits** when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` logic in `pick_regular_commit()`. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled. The regression is **urgent** and must be fixed before the next release.

> **Why it matters**: `git replay` is a new command (introduced in Git 2.54), and this regression could break workflows that rely on linearizing histories. The fix will likely involve restoring the `replayed_base` parameter or redesigning the replay logic to handle merges more robustly.

---

### `git cat-file --batch-command` reaches final readiness
Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries is **technically complete** and ready for merging. The series introduces a new `remote-object-info` command that lets clients request metadata (e.g., object size) for up to 10,000 objects in a single command, reducing network overhead. Key features:
- **Dynamic format validation**: Placeholders are validated based on server capabilities.
- **Security hardening**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.
The series is a **significant step forward for partial clone workflows**, though a **philosophical debate** over silent failures versus explicit errors remains unresolved.

> **Why it matters**: This feature enables **efficient metadata queries** for partial clones, reducing the need to fetch full objects. It’s a key enabler for large-scale monorepos where clients only need metadata (e.g., object sizes) for many objects.

---

### `git history squash` advances to v6
Harald Nordgren’s **4-patch series** implementing `git history squash` (folding a commit range into its oldest commit) reached **v6**, addressing all prior feedback. Key improvements:
- **Stricter input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`.
- **Multiple revision arguments**: Supports compound ranges (e.g., `@~3.. ^topic`).
- **`--reedit-message` template**: Aligns with `git rebase -i`’s squash behavior.
The series is **functionally complete** and ready for Junio’s final review.

> **Why it matters**: `git history squash` provides a **simpler alternative to `git rebase -i`** for linearizing histories, with predictable, all-or-nothing behavior. The v6 changes address edge cases and improve usability.

---

### CI hangs in `t5551`/`t5559` resolved
Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` was **merged**. The series:
1. Increases Apache’s `Timeout` directive from 300 to 600 seconds.
2. Isolates the expensive "many-tags" test case into a dedicated repository.
3. Packs refs after creating many tags (optional optimization).
The root cause was **Apache bug 70131**, which caused HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs.

> **Why it matters**: This resolves a **long-standing CI flake** that has plagued Git’s macOS CI for months. The fix is conservative (600 seconds is double the default but far below tested values) and aligns with the project’s preference for addressing flakes at the source.

---

## In brief

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**HTTPS proxy regression fix** -- Johannes Schindelin’s **2-line fix** for a regression in `set_curl_proxy_type()` (rejecting HTTPS proxy URLs) was **merged into `master`**. The fix restores the intended behavior of accepting HTTPS proxies.

**Reftable memory leak fix** -- Jeff King’s **4-line fix** for a memory leak in `reftable_writer_new` was **merged**. The leak was introduced in February 2025 and only recently flagged by Coverity.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue before the next release.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw. The series’ future may depend on Taylor’s availability after a role change.
- **`git history squash`**: Harald Nordgren’s **v6 series** is ready for Junio’s final review.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.

---

## The day’s texture
Today’s traffic was **dominated by landing series and regressions**, reflecting a **pre-release push** ahead of Git 2.55. The **reftable security hardening** and **`paint_down_to_common()` optimization** are particularly notable for their **long-term impact**—the former sets a new standard for security testing in Git, while the latter delivers a **rare 100-1000x performance improvement**. The **`git replay --linearize` regression** is a reminder that even well-tested features can have edge cases, and the urgency of the fix underscores the project’s commitment to stability.

The **`git cat-file --batch-command`** series reaching final readiness is a **GSoC success story**, demonstrating how structured mentorship can bring complex features to completion. Meanwhile, the **CI hang resolution** shows the project’s pragmatism in addressing flakes without resorting to workarounds.

Overall, today was a **high-impact day** with **lasting consequences** for Git’s performance, security, and usability.