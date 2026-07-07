# The Git Project Daily Digest
**2026/06/28 (UTC) – A Sunday of Regressions, Optimizations, and Final Touches**

The day’s **45 emails across 17 threads** painted a quiet but consequential Sunday: a **post-merge regression in `git replay --linearize`** dominated attention, while **performance optimizations, security hardening, and final polish** on long-running series rounded out the traffic. The tone was **technical and focused**, with no heated debates—just the steady hum of a project tying up loose ends before the next release.

---

## The day in brief
A **regression in `git replay --linearize`** stole the spotlight, as Johannes Schindelin identified a **silent commit-dropping bug** in the recently merged v5. Meanwhile, **Tian Yuchen’s `paint_down_to_common()` optimization** received its final approval, **Harald Nordgren’s `git history squash`** reached v6, and **Jeff King’s CI timeout fix** for macOS runners was posted. The day’s texture was **milestone-heavy**: several series reached **technical completion**, while one **urgent regression** demanded immediate follow-up.

---

## Notable threads

### `git replay --linearize` regression: silent commit dropping
**Headline:** *`git replay --linearize` drops commits when replaying a single branch with merges*
Johannes Schindelin reported a **critical regression** in the recently merged `git replay --linearize` (v5): when replaying a range like `master~2..master` with `--linearize --onto master~2`, the command **silently drops the first commit** ("Git 2.55-rc2") and replays only the tip. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was originally introduced to prevent this exact behavior. Schindelin’s test case—replaying three branches where one is a merge of the other two—illustrates the complexity of ensuring all replayed commits are linearized into a single topology.

**Why it matters:** This is a **post-merge regression** in a new feature, and the silent data loss makes it urgent. The fix will likely involve restoring the `replayed_base` logic or redesigning how multi-branch histories are handled. Expect a follow-up patch from Toon Claes early next week.

---

### `paint_down_to_common()` optimization: final approval
**Headline:** *Tian Yuchen’s 8-patch series optimizing merge-base calculations is fully approved*
Derrick Stolee gave the **final sign-off** on Tian Yuchen’s series optimizing `paint_down_to_common()` for one-sided histories. The series terminates merge-base walks early when one side’s commit queue is exhausted, delivering **100-1000x speedups** for asymmetric queries (e.g., repositories with import grafts). Key improvements in v4:
- **Fixed a regression** in patch 7/8 (unconditional BUG assertion).
- **Unified halt conditions** in the `paint_state` struct.
- **Added trace2 instrumentation** for performance metrics.
- **New technical document** (`paint-down-to-common.adoc`).

**Why it matters:** This is a **major performance win** for `git merge-base` in large repositories, with no known regressions. The series is now **ready for merging** into `next`.

---

### `git history squash` v6: stricter input validation
**Headline:** *Harald Nordgren’s `git history squash` series reaches v6 with support for multiple revision arguments*
Harald Nordgren posted **v6 of `git history squash`**, a new subcommand that folds a commit range into its oldest commit while preserving descendant history. The headline change: **support for multiple revision arguments** (e.g., `@~3.. ^topic`), addressing Junio’s usability concern about the earlier single-argument limitation. The series also adds **stricter input validation**, rejecting single-commit ranges and ranges with multiple bases. The `--reedit-message` flag now uses `git rebase -i`’s squash-message template, resolving a long-standing usability critique.

**Why it matters:** The series is **functionally complete** and ready for Junio’s final review. The shift to multiple revision arguments makes the command more flexible and aligns it with other `git history` subcommands.

---

### CI timeout fix for macOS runners
**Headline:** *Jeff King’s 3-patch series increases Apache’s `Timeout` to 600 seconds*
Jeff King posted a **3-patch series** addressing macOS CI hangs in `t5551` and `t5559`. The root cause: Apache’s default 300-second CGI timeout was being hit during `ls-refs` advertisements of 100,000 loose refs. The series:
1. **Increases Apache’s `Timeout` to 600 seconds** (patch 1/3).
2. **Isolates the expensive "many-tags" test case** into its own repository (patch 2/3).
3. **Packs refs after creating many tags** to reduce `ls-refs` time (patch 3/3).

**Why it matters:** This is a **low-risk, high-impact fix** for a persistent CI flake. The first patch alone resolves the issue, while the others are optimizations. Junio’s review suggests the series may be trimmed to just the first two patches.

---

### `USE_NSEC` Meson parity: modern Linux filesystems preserve nanoseconds
**Headline:** *Jeff King’s testing shows `USE_NSEC` is safe on modern Linux filesystems*
Jeff King’s follow-up to D. Ben Knoble’s `USE_NSEC` Meson parity patch revealed that **modern Linux filesystems (ext4, ext2, XFS, vfat, CIFS, NTFS, FUSE) preserve nanosecond timestamps correctly**, obsoleting historical "racy Git" concerns. His test methodology—writing a file, dropping caches, and re-reading timestamps—confirmed that nanoseconds survive a reload from disk. However, he also identified **interoperability risks** when mixing `USE_NSEC` and non-`USE_NSEC` implementations (e.g., Git and JGit), which could trigger stat-dirty re-reads.

**Why it matters:** The findings weaken the case for keeping `USE_NSEC` as a safety measure, but the interoperability caveats add complexity. The discussion now centers on whether the knob should be **deprecated, flipped to `true` by default, or made runtime-configurable**.

---

## In brief

**`git cat-file --batch-command`** -- Pablo Sabater’s **14-patch GSoC series** is technically complete and ready for merging. The series introduces a `remote-object-info` command for querying object metadata from remotes, with **dynamic format validation** and **680 lines of new tests**.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series aligns with the broader `the_repository` removal initiative.

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligning translation catalog generation with core Git’s quiet build conventions is **merged (gitk) or superseded (git-gui)**.

**HTTPS proxy regression** -- Johannes Schindelin’s **2-line fix** for a regression rejecting HTTPS proxy URLs was merged into `master`. The fix preserves the intended behavior of rejecting unsupported schemes while correctly handling HTTPS proxies.

**Reftable memory leak** -- Jeff King’s **4-line fix** for a memory leak in `reftable_writer_new` was merged, with a trivial conflict resolution noted.

**French translation update** -- Jiang Xin’s **l10n pull request** for Git 2.55.0 covers 11 languages and includes a mass typo-fix pass for French. The request is **ready for merging**.

---

## On the radar
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to restore the `replayed_base` logic or redesign multi-branch handling.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for a reachability filtering flaw that could incorrectly place reachable objects in cruft packs.
- **`git history squash`**: Harald Nordgren’s **v6 series** awaits Junio’s final review.
- **`USE_NSEC` default**: The discussion about flipping the default to `true` or making it runtime-configurable remains unresolved.