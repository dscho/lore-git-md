Here’s the daily digest for **2026/06/28** (Sunday), covering the Git mailing list’s key developments in a concise, front-page-style overview:

---

# **The day in brief**
**2026/06/28** saw **45 emails across 17 threads**, marking a **quiet but technically dense** day. The standout themes: **regressions surfacing in recently merged features** (`git replay --linearize`), **final polish on long-running series** (Pablo Sabater’s GSoC `git cat-file --batch-command`), and **performance optimizations landing** (Tian Yuchen’s `paint_down_to_common()`). The most urgent item: a **post-merge regression in `git replay --linearize`** that silently drops commits, requiring a follow-up patch.

---

## **Notable threads**

### **`git replay --linearize` regression: commits silently dropped**
**Thread**: *[PATCH 0/3] replay: introduce --linearize option* (v6)
**Author**: Toon Claes
**Status**: **Urgent follow-up needed**

Johannes Schindelin identified a **critical regression** in `git replay --linearize` (merged in v5): when replaying a single branch containing merge commits (e.g., `master~2..master --linearize --onto master~2`), the command **silently drops commits** instead of flattening the entire range. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact behavior. Schindelin’s test case (`master~2..master` with `--linearize --onto master~2`) now replays only the tip commit, omitting the first replayed commit ("Git 2.55-rc2").

### Key details

- **Root cause**: Loss of `replayed_base` tracking in `pick_regular_commit()`.
- **Impact**: Silent data loss during linearization of merge-containing branches.
- **Next steps**: Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign the multi-branch handling logic. The regression is **blocking for the next release**.

---

### **GSoC `git cat-file --batch-command` series reaches final readiness**
**Thread**: *[PATCH v15 0/13] cat-file: add --batch-command remote-object-info*
**Author**: Pablo Sabater
**Status**: **Technically complete; ready for merging**

Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries is now **feature-complete** and addresses all prior feedback. The series introduces a new `remote-object-info` command that lets clients request metadata (e.g., object size) for up to 10,000 objects in a single command, reducing network overhead for partial clone workflows. Key improvements in v15:
- **Dynamic format validation**: Placeholders (e.g., `%(objectsize)`) are validated based on server-advertised capabilities.
- **Security hardening**: Strict protocol v2 enforcement and input validation.
- **Test coverage**: 680 lines of new tests in `t1017-cat-file-remote-object-info.sh`.

**Open question**: A **philosophical debate** remains unresolved—should the client fail explicitly or continue silently when metadata is missing? Pablo prefers silent continuation (matching `git cat-file`’s behavior), while Karthik Nayak advocates for explicit errors. The discussion does not block merging.

---

### **`paint_down_to_common()` optimization lands**
**Thread**: *[PATCH v4 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories*
**Author**: Tian Yuchen
**Status**: **Merged into `next`**

Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for asymmetric histories was **fully approved and merged**. The series terminates merge-base walks early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories (e.g., repositories with import grafts). Key achievements:
- **100-1000x speedups** for asymmetric queries (e.g., `merge-base --all` across a 2.6M-commit monorepo drops from 3.67s to 5ms).
- **Regression fix**: Addressed an unconditional BUG assertion in patch 7/8.
- **Instrumentation**: Added trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).

The optimization is a **major win for `git merge-base` performance** in asymmetric histories, with no known regressions.

---

### **`git history squash` series reaches v6**
**Thread**: *[PATCH v6 0/4] history: introduce `squash` subcommand*
**Author**: Harald Nordgren
**Status**: **Ready for Junio’s final review**

Harald Nordgren’s **4-patch series** introducing `git history squash` (folding a commit range into its oldest commit while preserving descendant history) reached **v6**, addressing all prior feedback. Key improvements:
- **Stricter input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (since the marker’s target cannot lie inside the range).
- **Support for multiple revision arguments**: Now accepts compound ranges (e.g., `@~3.. ^topic`).
- **`--reedit-message` template alignment**: Matches `git rebase -i`’s squash behavior, with `fixup!` messages fully commented out and `squash!`/`amend!` bodies retained.

### Open questions

- Should `--reedit-message` (or `--edit`) be the default? Phillip Wood advocates for this as a commit hygiene measure.
- **Recoverability concerns**: Matt Hunter and Phillip Wood note that `git reset --hard` may not suffice if `--update-refs` moves other branches, highlighting a potential usability gap.

---

### **CI hangs in `t5551`/`t5559` resolved**
**Thread**: *macOS CI hang in t5551/t5559 – root cause and fix*
**Author**: Jeff King (Peff)
**Status**: **Ready for merging**

Jeff King’s **3-patch series** addressing macOS CI hangs in `t5551` and `t5559` is **ready for merging**. The series:
1. **Increases Apache’s `Timeout` directive** from 300 to 600 seconds (patch 1/3).
2. **Isolates the expensive "many-tags" test case** into a dedicated repository (patch 2/3).
3. **Packs refs after creating many tags** to reduce `ls-refs` advertisement time (patch 3/3).

The series directly targets Apache bug 70131, which caused HTTP/2 stalls or `curl 18` mid-transfer aborts during `ls-refs` advertisements of 100,000 loose refs. Junio C Hamano noted a **minor stylistic nit** (non-bare repository initialization in patch 2/3) but confirmed the fix is otherwise correct.

---

## **In brief**

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass alongside the usual string updates. The pull request covers 11 languages and is **ready for merging**.

**`git history --reword` fix** -- Junio’s **bugfix patch** addresses a file stream leak in `git history --reword` that could cause problems on Windows. The fix is **merge-ready**.

---

## **On the radar**
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for a reachability filtering flaw.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review.
- **`--track=fetch` debate**: Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on workflow concerns.