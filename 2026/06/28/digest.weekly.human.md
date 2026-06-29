# The Git Project Weekly Digest
**2026/06/22 -- 2026/06/28**

## The period in brief
This was a **high-volume, technically dense week** (515 emails across 118 threads on 6 active days) that saw **major architectural series reach completion**, **security-hardening efforts land**, and **long-running usability improvements finalize**. The standout developments: **Patrick Steinhardt’s ref backend lazy-loading refactor merged**, **Pablo Sabater’s `git cat-file --batch-command` security series reached final readiness**, and **Tian Yuchen’s `paint_down_to_common()` optimization landed after a critical regression was caught and fixed**. The week also featured **post-merge regressions in `git replay --linearize`** and **philosophical debates** over `--track=fetch` and ODB abstraction trade-offs.

---

## Key developments

### Ref backend lazy-loading refactor lands
Patrick Steinhardt’s **11-patch series** modernizing Git’s reference backend infrastructure was **merged into `next`**, resolving recursive initialization issues caused by `includeif.onbranch` conditions. The redesign defers write-config parsing until the first write operation, eliminating early config reads that could trigger recursion. The series touches 34 files, including core ref backends (`files-backend.c`, `reftable-backend.c`), and introduces a recursion guard in `get_main_ref_store()`. This is a **foundational improvement** for ref backend modularity, with no user-visible behavior changes unless `includeif.onbranch` is used. The series is poised to graduate to `master` in the next integration cycle.

---

### `git cat-file --batch-command` security series reaches final readiness
Pablo Sabater’s **14-patch GSoC series** implementing `git cat-file --batch-command` for remote object metadata queries (e.g., object sizes) is now **technically complete and ready for merging**. The series introduces a new `remote-object-info` command that lets clients request metadata for up to 10,000 objects in a single command, reducing network overhead for partial clone workflows. Key improvements in v14 include **dynamic capability-based validation of format placeholders**, **memory safety fixes**, and **comprehensive test coverage** (680 lines in `t1017-cat-file-remote-object-info.sh`). The series is **security-hardened**, with strict protocol v2 enforcement and input validation. The only unresolved question is **error-handling philosophy**: should the client fail explicitly when metadata is missing, or continue silently (matching local `git cat-file` behavior)? Pablo defends silent continuation as consistent and pragmatic, while Karthik Nayak argues it complicates future extensibility.

---

### `paint_down_to_common()` optimization lands after regression fix
Tian Yuchen’s **8-patch series** optimizing `paint_down_to_common()` for one-sided histories was **fully approved and merged**, delivering **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts). The series terminates early when one side’s commit queue is exhausted, eliminating unnecessary traversal of large one-sided histories. A **critical regression** in patch 7/8 (an unconditional BUG assertion) was caught by the test suite and fixed in v4. The series includes **trace2 instrumentation**, **dead-code removal**, and a new technical document (`paint-down-to-common.adoc`). This is a **major performance win** for `git merge-base` in repositories with asymmetric histories.

---

### Reftable security hardening: 11 patches fully reviewed
Patrick Steinhardt’s **11-patch series** hardening the reftable backend against corrupted files is now **fully reviewed and ready for `next`**. The series fixes **OOB reads/writes, NULL pointer dereferences, and `abort()` calls** in the reftable parser, and adds a **libFuzzer-based fuzzer** to prevent regressions. The fixes are **internal-only** (no user-facing changes) and critical for the reftable backend’s stability. The fuzzing infrastructure is a **long-term win** for security, and the series is expected to graduate to `master` shortly.

---

### `git branch --delete-merged` v18: implementation-complete
Harald Nordgren’s **18th iteration** of the `git branch --delete-merged` series is now **code-complete and ready for final review**. The series adds a safe, automated way to clean up merged local branches, with **stacked-branch protection** (aborts if a branch is used as an upstream for another branch) and **per-branch opt-out** (`branch.<name>.deleteMerged=false`). The `--dry-run` flag matches real deletion output, and the `--forked` filter composes with `--merged`/`--no-merged`. The series is **well-tested** (497-line test suite) and addresses all prior feedback. Junio’s final assessment is pending.

---

### `git replay --linearize` post-merge regression: silent commit dropping
Johannes Schindelin reported a **post-merge regression** in `git replay --linearize` (v5), where the command **silently drops commits** when replaying a single branch containing merge commits. In a real-world example (`master~2..master`), only the tip commit is replayed, and the “Git 2.55-rc2” commit is lost. The regression stems from the removal of the `replayed_base` logic in `pick_regular_commit()`. Toon Claes is expected to send a follow-up patch to restore the mechanism or redesign how multi-branch histories are handled. This is a **critical regression** that needs urgent attention before the next release.

---

### ODB abstraction: `struct object_info` refactoring approved
Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use a `source` field (replacing the coarse `whence` enum) received **Junio’s conceptual approval** ("Great"). The series enables **multi-source object resolution** and is a **foundational step** for pluggable ODB backends. Key changes include threading `struct odb_source_packed *source` through `packed_object_info()` and adding documentation for `struct object_info` fields. The series is **ready for substantive review** and has no objections yet.

---

### `git repack --geometric --cruft` RFC: correctness flaw diagnosed
Taylor Blau’s **10-patch RFC** to combine `--geometric` and `--cruft` repack modes hit a **correctness snag**: Junio identified a flaw in the two-phase traversal logic of `--stdin-packs=follow-reachable`, where **unreachable tags and objects** may be incorrectly retained. The current implementation marks *all* tag objects in included packs as reachable, even if they’re not referenced by any ref. Taylor proposed reversing the traversal order (walk from refs first, then mark objects in included packs), but no concrete fix has been submitted. The series remains in **RFC limbo** until this is resolved.

---

## In brief

**`git history squash` v6** -- Harald Nordgren’s series to fold commit ranges into a single commit is **functionally complete**, with stricter input validation, support for multiple revision arguments, and the `--reedit-message` flag. The series is ready for Junio’s final review.

**`git log -L` range-scoped diff stat** -- Karthik Nayak’s 7-patch series extending `git log -L` to support `--stat`, `--check`, and `-G` pickaxe operations is **complete and uncontroversial**. The series adds range-scoped diff stat formats and whitespace checking.

**`excludes_file` libification** -- Tian Yuchen’s patch moving the global `excludes_file` into `struct repo_config_values` is **ready for `next`** after resolving a guardrail debate. The series is part of the **libification effort** to eliminate global state.

**`git history --reword` file leak fix** -- Junio’s **bugfix patch** addresses a file stream leak in `git history --reword` that could cause problems on Windows. The fix consolidates file handling and is **merge-ready**.

**`--track=fetch` workflow debate** -- Harald Nordgren’s `--track=fetch` feature for `git checkout`/`git switch` remains stalled on **philosophical concerns** about network operations during checkout. Junio’s reservations about "blind updates" have not been addressed.

**CI hangs in `t5551`/`t5559`** -- Jeff King’s 3-patch series increases Apache’s `Timeout` directive to 600 seconds and isolates the expensive “many-tags” test case. The fix is **ready for merging**.

**French translation update** -- Jean-Noël Avila’s update for Git 2.55.0 includes a mass typo-fix pass and is **ready for merging**.

**`greplint.pl` exposes test-suite fragility** -- Junio demands an audit of all `# lint-ok:` comments to ensure they do not hide latent test bugs. The discussion highlights a tension between automation and diagnostic rigor.

---

## Looking ahead
- **`git replay --linearize` regression**: Toon Claes must send a follow-up patch to fix the silent commit dropping issue before the next release.
- **`git repack --geometric --cruft`**: Taylor Blau’s RFC needs a fix for the reachability filtering flaw. The series is a **high-impact workflow improvement** for large repositories.
- **`--track=fetch`**: The philosophical debate over network operations during checkout remains unresolved. Junio’s final decision is pending.
- **ODB abstraction**: Patrick Steinhardt’s `struct object_info` refactoring is ready for substantive review. The series is foundational for pluggable backends.
- **Git 2.55.0**: The release is imminent, with only regression fixes and late topics expected to graduate before the final release. Junio’s “What’s cooking” report (#09) signals deep feature-freeze.