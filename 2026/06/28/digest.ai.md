Here’s the daily digest for **2026/06/28**, covering the Git mailing list’s key developments in a concise, front-page-style overview:

---

# The Git Daily Digest
**2026/06/28: A Day of Performance Wins, Regression Fixes, and Security Landings**

### The day in brief
A **moderately busy Sunday** (45 emails, 17 threads) saw **critical performance optimizations merged**, a **post-merge regression in `git replay` exposed**, and **security hardening for reftable finalized**. The standout: Tian Yuchen’s `paint_down_to_common()` optimization (100-1000x speedups for asymmetric histories) and Patrick Steinhardt’s reftable fuzzing series both landed, while a late-breaking bug in `git replay --linearize` threatens to delay its graduation. L10n updates and CI fixes rounded out the day.

---

## Notable threads

### `paint_down_to_common()` optimization lands
**Thread**: [commit-reach: optimize `paint_down_to_common()` for one-sided histories](https://lore.kernel.org/git/20260628122538.GA12345@tianyuchen/)
**Author**: Tian Yuchen (with Kristofer Karlsson)
**Status**: **Merged into `next`** (v4, 8 patches)
**Impact**: **100-1000x speedup** for `git merge-base` in asymmetric histories (e.g., repositories with import grafts).

The series teaches Git’s merge-base machinery to terminate early when one side of the history is exhausted, eliminating unnecessary traversal of large one-sided histories. Key changes:
- **Regression fix**: Restored the `min_generation` guard to a BUG assertion (patch 7/8).
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical doc (`paint-down-to-common.adoc`).
- **Test coverage**: Deterministic step-count assertions for all commit-graph modes.

**Why it matters**: This is a **major performance win** for monorepos and shallow histories, with no known regressions. The optimization is now **enabled by default** and will ship in Git 2.55.

---

### `git replay --linearize` regression surfaces
**Thread**: [PATCH 0/3] replay: introduce --linearize option](https://lore.kernel.org/git/20260628122013.GA6789@schindelin.local/)
**Author**: Johannes Schindelin
**Status**: **Urgent follow-up needed** (regression in v5)
**Impact**: `git replay --linearize` **silently drops commits** when replaying a single branch containing merge commits.

The bug was introduced in v5 when the `replayed_base` parameter was removed from `pick_regular_commit()`. Schindelin’s report includes a **reproducible test case** (`master~2..master` with `--linearize --onto master~2`) and traces the issue to the loss of parent tracking. Toon Claes is expected to send a fix **before the next release**.

**Why it matters**: The regression undermines the feature’s core promise of predictable history flattening. If unresolved, it could delay `git replay`’s graduation from `next`.

---

### Reftable security hardening finalized
**Thread**: [Re: [PATCH v11 00/11] reftable: fixes for corrupted files](https://lore.kernel.org/git/20260628090314.GA3456@peff/)
**Author**: Patrick Steinhardt
**Status**: **Merged into `next`** (11 patches)
**Impact**: Fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in Git’s reftable backend.

The series includes:
- **Fuzzing infrastructure**: A libFuzzer-based fuzzer now runs for 2+ hours in CI without finding new issues.
- **Critical fixes**: Patches 5/11 (heap-buffer-overflow) and 11/11 (OOB read on truncated tables) address the most severe bugs.
- **Memory leak fix**: Jeff King’s follow-up patch (merged) plugs a leak in `reftable_writer_new`.

**Why it matters**: The reftable backend is **critical for Git’s future**, and this series sets a new bar for security testing. The fuzzer will now run continuously to catch regressions.

---

### `git cat-file --batch-command` reaches final readiness
**Thread**: [PATCH v15 00/13] cat-file: add --batch-command remote-object-info](https://lore.kernel.org/git/20260628225522.GA8901@pablo/)
**Author**: Pablo Sabater
**Status**: **Technically complete** (14 patches, v15)
**Impact**: Enables **remote object metadata queries** (e.g., object size) without full downloads.

The series introduces a new `remote-object-info` command for `git cat-file --batch-command`, allowing clients to request metadata for up to 10,000 objects in a single command. Key features:
- **Dynamic format validation**: Placeholders are validated based on server capabilities.
- **Security hardening**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.

**Open question**: A **philosophical debate** over silent failures vs. explicit errors remains unresolved, but the series is otherwise **ready for merging**.

---

### CI hangs in `t5551`/`t5559` resolved
**Thread**: [PATCH 0/3] macOS CI hang in t5551/t5559 – root cause and fix](https://lore.kernel.org/git/20260628080710.GA1234@peff/)
**Author**: Jeff King
**Status**: **Ready for merging** (3 patches)
**Impact**: Eliminates **HTTP/2 stalls and `curl 18` mid-transfer aborts** in CI.

The series:
1. **Increases Apache’s `Timeout` directive** from 300 to 600 seconds (patch 1/3).
2. **Isolates the expensive "many-tags" test case** into a dedicated repository (patch 2/3).
3. **Packs refs after creating many tags** to reduce `ls-refs` advertisement time (patch 3/3).

**Why it matters**: The fix addresses a **persistent CI flake** caused by Apache bug 70131, ensuring reliable test runs on macOS and Linux.

---

## In brief

**`git history squash`** -- Harald Nordgren’s **v6 series** (4 patches) is **functionally complete**, with stricter input validation and support for multiple revision arguments (e.g., `@~3.. ^topic`). Ready for Junio’s final review.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a **mass typo-fix pass** alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligns translation catalog generation with core Git’s quiet build conventions. The patches are **merged (gitk) or superseded (git-gui)**.

---

## On the radar

**`git repack --geometric --cruft`** -- Taylor Blau’s **10-patch RFC** faces a **correctness issue** in its reachability filtering. Junio identified that the two-phase traversal may retain **unreachable tags and objects**, undermining the separation of reachable/unreachable objects.

**ODB abstraction** -- Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use a `source` field (replacing the `whence` enum) was **conceptually approved** by Junio. Awaiting substantive review.

**`--track=fetch` debate** -- Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on workflow concerns, despite being technically complete.

---

## The week ahead
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch **before the next release**.
- **`git repack --geometric --cruft`**: Taylor Blau needs to address the reachability filtering flaw.
- **`git history squash`**: Harald Nordgren’s **v6 series** awaits Junio’s final review.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring needs deeper review.