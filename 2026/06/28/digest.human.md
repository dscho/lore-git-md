# The Git Project Daily Digest
**2026/06/28 (Sunday) – A day of regressions, refinements, and readiness**

This was a **moderate-volume Sunday** (45 emails across 17 threads), with a **technical focus on regressions, final refinements, and readiness assessments**. The standout developments: a **critical regression in `git replay --linearize`** resurfaced, **Harald Nordgren’s `git history squash` series reached v6**, and **Junio fast-tracked a bugfix for HTTPS proxy handling**. The day also saw **final approvals for performance optimizations** and **CI fixes**, alongside **lingering debates over `USE_NSEC` and reftable tombstone handling**.

---

## Notable threads

### **`git replay --linearize` regression: silent commit dropping**
**Thread**: [replay: introduce --linearize option](https://lore.kernel.org/git/20260604074552.12345-1-toon@iotcl.com/)

Johannes Schindelin **reported a regression in v5 of `git replay --linearize`** that silently drops commits when replaying a single branch containing merge commits. The issue stems from the removal of the `replayed_base` parameter, which was originally introduced to prevent this exact behavior. Schindelin’s test case—replaying `master~2..master` with `--linearize --onto master~2`—demonstrates the problem: only the tip commit is replayed, omitting the first commit in the range ("Git 2.55-rc2").

### Key details

- **Root cause**: The `replayed_base` parameter ensured all replayed commits were linearized into a single topology. Without it, the replay machinery loses track of the correct parent for commits following a merge.
- **Impact**: The regression affects users replaying single branches with merges, a common workflow for history rewriting.
- **Status**: Toon Claes is expected to send a follow-up patch to restore the `replayed_base` logic or redesign the multi-branch handling. The fix is **urgent** and must land before the next release.

**Why it matters**: This is a **post-merge regression** in a recently introduced feature, and the silent commit dropping could lead to data loss if users don’t verify the output. The thread underscores the importance of **regression testing for edge cases** in new commands.

---

### **`git history squash` reaches v6: stricter validation, multi-argument support**
**Thread**: [PATCH v6 0/4] replay: introduce `git history squash`](https://lore.kernel.org/git/pull.1723.git.1719578949.gitgitgadget@gmail.com/)

Harald Nordgren **posted v6 of the `git history squash` series**, addressing all prior feedback and adding **stricter input validation** and **support for multiple revision arguments** (e.g., `@~3.. ^topic`). The series folds a commit range into its oldest commit while preserving descendant history, offering a simpler alternative to `git rebase -i` for linearizing histories.

### Key improvements in v6

- **Input validation**: Rejects single-commit ranges, empty ranges, and ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **Multi-argument support**: Parses revision ranges like `git rev-list`, enabling compound ranges (e.g., `@~3.. ^topic`).
- **Merge commit handling**: Explicitly rejects merges with external parents but allows fully contained merges (all parents within the range).
- **Ref safety**: Rejects operations where any ref points to a commit inside the squashed range, with an advice message suggesting `--update-refs=head`.

### Open questions

- **Default behavior**: Phillip Wood and Matt Hunter argue that `--reedit-message` (or `--edit`) should be the default to encourage commit hygiene. Harald has not yet addressed this.
- **Recoverability**: The series lacks a robust undo mechanism for `--update-refs` operations, though this is a broader Git limitation (not specific to `squash`).

**Status**: The series is **technically complete** and ready for Junio’s final review. The **interface design debate** (whether to mirror `git rebase --rebase-merges=<mode>`) is resolved in favor of the standalone `--linearize` flag, with explicit justification in the commit message.

---

### **HTTPS proxy regression fix fast-tracked for Git 2.55**
**Thread**: [http: fix regression rejecting HTTPS proxy URLs](https://lore.kernel.org/git/20260627171756.7890-1-johannes.schindelin@gmx.de/)

Junio C Hamano **fast-tracked Johannes Schindelin’s fix** for a regression in HTTPS proxy URL handling. The bug, introduced in commit `663d7abe07ea` (http: reject unsupported proxy URL schemes), incorrectly rejected HTTPS proxy URLs (`https://...`) despite the code already supporting them. The fix is a **two-line change** in `http.c` that adds a `return 0` for the HTTPS branch in `set_curl_proxy_type()`.

### Key details

- **Root cause**: The HTTPS branch fell through to a `return -1` intended for unknown schemes.
- **Impact**: Users with HTTPS proxies could not clone or fetch from repositories.
- **Status**: The patch is **merged into `master`** ahead of Git 2.55.

**Why it matters**: This is a **critical regression fix** with minimal risk, and Junio’s fast-tracking reflects the urgency of ensuring HTTPS proxy support works out of the box.

---

### **`paint_down_to_common()` optimization fully approved**
**Thread**: [PATCH v4 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories](https://lore.kernel.org/git/pull.1724.git.1719583542.gitgitgadget@gmail.com/)

Derrick Stolee **gave final approval** to Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories. The series terminates merge-base walks early when one side’s commit queue is exhausted, delivering **100-1000x speedups** for asymmetric queries (e.g., repositories with import grafts). The optimization is now **fully reviewed and ready for merging**.

### Key achievements

- **Regression fix**: Addressed an unconditional BUG assertion in patch 7/8.
- **Code clarity**: Unified halt conditions in the `paint_state` struct.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).

**Status**: The series is **merged into `next`** and will ship in Git 2.55.

---

### **CI hangs in `t5551`/`t5559`: Apache timeout fix merged**
**Thread**: [macOS CI hang in t5551/t5559 – root cause and fix](https://lore.kernel.org/git/20260620160024.GA12345@peff.net/)

Junio C Hamano **merged Jeff King’s 2-patch series** addressing macOS CI hangs in `t5551` and `t5559`. The root cause was **Apache’s `mod_http2` bug 70131**, which triggered either a mid-transfer abort (`curl 18`) or an indefinite HTTP/2 stall when the server-side 300-second timeout was hit. The fix:
1. **Increases Apache’s `Timeout` directive from 300 to 600 seconds** in `t/lib-httpd/apache.conf`.
2. **Isolates the expensive "many-tags" test case** into a dedicated repository to avoid interference.

### Key details

- **Impact**: Resolves CI flakes on GitHub’s macOS runners and similar environments.
- **Status**: The series is **merged into `master`**.

**Why it matters**: This is a **low-risk, high-impact fix** for a persistent CI issue, and the timeout increase is conservative (600 seconds is far below the 3600-second value tested by Michael Montalbo).

---

### **`USE_NSEC` Meson parity: testing reveals obsolescence on Linux**
**Thread**: [PATCH] meson: add `nanosec` option to mirror Autotools `USE_NSEC`](https://lore.kernel.org/git/20260614175217.5678-1-ben.knoble@duke.edu/)

Jeff King’s **testing revealed that `USE_NSEC` is no longer problematic on modern Linux filesystems** (ext4, ext2, vfat, CIFS, NTFS, FUSE). His findings suggest the original "racy Git" concerns (documented in `Documentation/technical/racy-git.adoc`) are obsolete, as nanosecond timestamps are preserved correctly. However, he also identified **interoperability risks** when mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit), which could trigger stat-dirty re-reads.

### Key proposals

- **Brian M. Carlson**: Flip the default to `true` for most users, with a config knob for edge cases.
- **Patrick Steinhardt**: Always compile nanosecond support into Git (when the platform allows it) and expose it via a runtime config knob (e.g., `core.useNsec`).
- **D. Ben Knoble**: Proceed with the original Meson parity patch unless `USE_NSEC` is deprecated entirely.

**Status**: The original patch is **queued for merging**, but the broader discussion about the knob’s viability remains unresolved.

---

### **Reftable tombstone performance: patch reconsidered**
**Thread**: [macOS CI hang in t5551/t5559 – root cause and fix](https://lore.kernel.org/git/20260620160024.GA12345@peff.net/)

Kristofer Karlsson’s **patch fixing quadratic-time behavior in the reftable backend** is now **under reconsideration** due to limited real-world impact. The patch exposes tombstones to iterator bounds checks, reducing runtime from O(n²) to O(n) for bulk ref deletion/re-creation workflows. However, Patrick Steinhardt noted that the fix’s impact is negligible when compaction is enabled, and the added code complexity may not be justified.

### Key details

- **Reproducible scenario**: Bulk ref deletion without compaction (e.g., `git for-each-ref | git update-ref --stdin`).
- **Benchmark data**: 8,000 refs: 1.8s → 0.145s (100x speedup); 16,000 refs: 7.1s → 0.258s.
- **Status**: The patch remains in draft form ([GitGitGadget PR #2166](https://github.com/gitgitgadget/git/pull/2166)) and may be abandoned unless new benchmark data demonstrates broader value.

---

## In brief

**`greplint.pl` regression**: Junio C Hamano **identified a critical oversight** in the `greplint.pl` series: the proposed `test_grep` enhancement to error on missing files is redundant because `test_grep` already validates file existence. The discussion now centers on **why the existing safeguard fails to catch the bug in `t3420-rebase-autostash.sh`**.

**`git branch --set-upstream-to` usability fix**: Junio **queued Harald Nordgren’s v3 series** improving error messages for common command-line mistakes (e.g., `git branch --set-upstream-to=origin main`). The series is **ready for merging**.

**`gitk`/`git-gui` quiet builds**: Junio **acknowledged Johannes Sixt’s update** that the patches are in his testing branches and will be proposed for upstream Git inclusion in the coming weeks.

**`excludes_file` libification**: Tian Yuchen and Junio **resolved the guardrail debate**, agreeing to remove the redundant `repo->initialized` check from `repo_excludes_file()`. The series is **ready for `next`**.

**Reftable memory leak fix**: Junio **merged Jeff King’s fix** for a memory leak in `reftable_writer_new`, resolving a conflict with a recent commit.

**`git ls-remote` hang**: Steffen Nurpmeso **reported that the original busy-loop symptom has disappeared**, but `git ls-remote` now fails with a low-speed timeout under constrained network conditions. No root cause identified.

**Git 2.55.0 l10n pull request**: Jiang Xin **requested merging updates for 11 languages**, including a mass typo-fix pass for French. The pull request is **ready for merging**.

---

## On the radar

- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to restore the `replayed_base` logic or redesign multi-branch handling.
- **`git history squash` default behavior**: Phillip Wood and Matt Hunter argue that `--reedit-message` (or `--edit`) should be the default. Harald has not yet addressed this.
- **`USE_NSEC` runtime configuration**: Patrick Steinhardt’s proposal to make `USE_NSEC` runtime-configurable remains unresolved. The discussion hinges on the default value for `core.useNsec` and interoperability risks.
- **Reftable tombstone performance**: Kristofer Karlsson’s patch may be abandoned unless new benchmark data demonstrates broader value.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw. The series’ future may depend on Taylor’s availability after a role change.