# The Git Project Daily Digest
**2026/06/28 (Sunday) – A day of regressions, optimizations, and final readiness**

The day’s **45 emails across 17 threads** carried a **technical focus**, with **regressions in `git replay --linearize` and `greplint.pl`** demanding urgent attention, **performance optimizations landing**, and **multiple series reaching final readiness**. The standout developments: **Johannes Schindelin’s regression report in `git replay --linearize`** (silent commit dropping), **Junio’s follow-up on `greplint.pl`’s latent bug preservation**, and **Derrick Stolee’s final approval of Tian Yuchen’s `paint_down_to_common()` optimization** (100-1000x speedups). The day also saw **Harald Nordgren’s `git history squash` series post v6**, **Jeff King’s CI hang fixes land**, and **l10n updates for Git 2.55.0** submitted.

---

## Notable threads

### **`git replay --linearize` regression: silent commit dropping**
**Thread**: [replay: introduce --linearize option](https://lore.kernel.org/git/20260604074552.12345-1-toon@iotcl.com/)
**Author**: Johannes Schindelin
**Impact**: **Urgent** – the regression silently drops commits when replaying a single branch containing merge commits (e.g., `master~2..master` with `--linearize --onto master~2`).

Johannes Schindelin reported a **regression in v5 of `git replay --linearize`** where the command incorrectly replays only the tip commit, omitting earlier commits in the range. The issue stems from the removal of the `replayed_base` parameter in `pick_regular_commit()`, which was introduced to prevent this exact behavior. Schindelin’s test case (`master~2..master` with `--linearize --onto master~2`) now replays only "Git 2.55-rc2" instead of both "Git 2.55-rc2" and "Git 2.55-rc1". The regression is **blocking** and must be fixed before the next release.

### Key details

- **Root cause**: Removal of `replayed_base` in `pick_regular_commit()`.
- **Test case**: `git replay --linearize --onto master~2 master~2..master`.
- **Expected**: Both commits replayed.
- **Actual**: Only the tip commit replayed.
- **Files**: `replay.c` (core logic), `t3650-replay-basics.sh` (test coverage).
- **Status**: **Urgent follow-up needed** from Toon Claes.

---

### **`greplint.pl`: latent bug preservation and automation risks**
**Thread**: [PATCH 0/3] replay: introduce greplint.pl](https://lore.kernel.org/git/20260608183718.56789-1-michael.montalbo@gmail.com/)
**Author**: Junio C Hamano
**Impact**: **High** – the linter’s `# lint-ok:` exemptions may preserve latent bugs, and the tool’s automation risks masking test suite flaws.

Junio C Hamano’s follow-up in the `greplint.pl` thread exposed a **critical oversight**: the linter’s `# lint-ok:` exemptions can preserve latent bugs by bypassing `test_grep`’s file-existence checks. The specific case (`t3420-rebase-autostash.sh`) involves a test that greps a file (`file3`) that *should not exist* after `git rebase --quit`. The linter’s exemption preserves the bug because `test_grep`’s `test -f` check is skipped. Junio’s emails shift the discussion from tooling improvements to **test correctness**, highlighting a tension between automation and diagnostic rigor.

### Key details

- **Root cause**: `# lint-ok:` exemptions bypass `test_grep`’s file-existence checks.
- **Example**: `! grep dirty file3` in `t3420-rebase-autostash.sh` is invalid because `file3` should not exist.
- **Correct fix**: `test_path_is_missing file3` (Gábor’s 2021 patch).
- **Open question**: Should the linter’s conversion logic flag assertions on files that *should not exist*?
- **Status**: **Design debate ongoing**; Junio’s tone suggests the `# lint-ok:` mechanism may need rethinking.

---

### **`paint_down_to_common()` optimization lands**
**Thread**: [PATCH v4 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories](https://lore.kernel.org/git/20260620103653.7890-1-kristofer.karlsson@gmail.com/)
**Author**: Kristofer Karlsson (co-signed by Tian Yuchen)
**Impact**: **High** – 100-1000x speedups for asymmetric merge-base queries (e.g., repositories with import grafts).

Derrick Stolee’s **final approval** of Tian Yuchen’s 8-patch series marks a **major performance win** for `git merge-base`. The series teaches `paint_down_to_common()` to terminate early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories. Key improvements:
- **Regression fix**: Restored the `min_generation` guard to the BUG assertion (patch 7/8).
- **Code clarity**: Unified halt conditions in `struct paint_state`.
- **Instrumentation**: Trace2 metrics for "commits walked" and hyperfine benchmarks.
- **Documentation**: New technical document (`paint-down-to-common.adoc`).

### Key details

- **Performance**: 3.67s → 5ms for a 2.6M-commit monorepo.
- **Files**: `commit-reach.c`, `Documentation/technical/paint-down-to-common.adoc`, test scripts.
- **Status**: **Fully approved and ready for merging**.

---

### **`git history squash` v6: final readiness**
**Thread**: [PATCH v6 0/4] history: introduce `squash` subcommand](https://lore.kernel.org/git/20260614175217.12345-1-haraldnordgren@gmail.com/)
**Author**: Harald Nordgren
**Impact**: **Medium** – efficient commit range folding with `--reedit-message` and `--update-refs`.

Harald Nordgren posted **v6 of `git history squash`**, addressing all prior feedback:
- **Input validation**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`.
- **Template alignment**: Adopts `git rebase -i`’s squash-message template.
- **CLI reordering**: Moves dashed options before `<revision-range>`.
- **Documentation**: Clarifies merge limitations and feature overview.

### Key details

- **Behavior**: Folds a range into its oldest commit while preserving descendants.
- **Ref safety**: Rejects operations with interior refs by default; advises `--update-refs=head`.
- **Files**: `builtin/history.c`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
- **Status**: **Ready for Junio’s final review**.

---

### **CI hangs in `t5551`/`t5559` resolved**
**Thread**: [macOS CI hang in t5551/t5559 – root cause and fix](https://lore.kernel.org/git/20260620160024.56789-1-peff@peff.net/)
**Author**: Jeff King
**Impact**: **High** – resolves persistent CI flakes on macOS and Linux.

Jeff King’s **3-patch series** addressing Apache bug 70131 (HTTP/2 stalls) is **ready for merging**:
1. **Patch 1/3**: Increases Apache’s `Timeout` directive to 600 seconds.
2. **Patch 2/3**: Isolates the expensive "many-tags" test case.
3. **Patch 3/3**: Packs refs after creating many tags (marginal optimization).

### Key details

- **Root cause**: Apache’s 300-second CGI timeout hit during `ls-refs` advertisements.
- **Files**: `t/lib-httpd/apache.conf`, `t/t5551-http-fetch-smart.sh`.
- **Status**: **Ready for merging**; Junio noted a minor stylistic nit in patch 2/3.

---

## In brief

**`git cat-file --batch-command`** -- Pablo Sabater’s **14-patch GSoC series** is **technically complete** and ready for merging. The series introduces `remote-object-info` for metadata queries (e.g., object size) with dynamic format validation. **Open question**: Should the client fail explicitly or continue silently when metadata is missing?

**`git branch --set-upstream-to` usability** -- Harald Nordgren’s **v3 series** is **merged into `next`**. The patches improve error messages for missing slashes (e.g., `origin main` → `origin/main`) and slash-as-space mistakes (e.g., `git push origin/main`).

**`gitk`/`git-gui` quiet builds** -- Harald Nordgren’s **2-patch series** aligns translation catalog generation with core Git’s quiet build conventions. The patches are **merged (gitk) or superseded (git-gui)**.

**`USE_NSEC` Meson parity** -- D. Ben Knoble’s patch adding a `nanosec` Meson option is **queued**, but Jeff King’s testing suggests the knob is obsolete on modern Linux. The discussion now centers on whether the default should flip to `true`.

**`excludes_file` libification** -- Tian Yuchen’s **2-patch series** moving the global `excludes_file` variable into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate.

**l10n updates for Git 2.55.0** -- Jiang Xin’s pull request updates `.po` files for 11 languages and documents a leadership change in the Simplified Chinese team. **Ready for merging**.

**HTTPS proxy regression** -- Johannes Schindelin’s **bugfix patch** is **merged into `master`**. The patch corrects a control-flow error in `set_curl_proxy_type()` that incorrectly rejected HTTPS proxy URLs.

---

## On the radar

**`git replay --linearize` regression** -- Toon Claes must send a follow-up patch to restore the `replayed_base` logic or redesign multi-branch handling. **Urgent**.

**`greplint.pl` design** -- Junio’s follow-up suggests the `# lint-ok:` mechanism may need rethinking to avoid preserving latent bugs. **High priority**.

**`git repack --geometric --cruft`** -- Taylor Blau’s RFC needs a fix for the reachability filtering flaw. **Medium priority**.

**`git history squash`** -- Harald Nordgren’s **v6 series** awaits Junio’s final review. **Medium priority**.

**ODB abstraction** -- Patrick Steinhardt’s `struct object_info` refactoring awaits substantive review. **Long-term priority**.