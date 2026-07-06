## The day in brief

**2026-06-26** was a busy day on the Git mailing list, with **86 emails across 17 threads**. The most significant developments were the **final approval of Patrick Steinhardt’s ref backend refactoring** (now merged to `next`), the **v5 release of Toon Claes’s `git replay --linearize` series** (addressing all prior feedback), and a **critical regression identified in Tian Yuchen’s merge-base optimization** (blocking its progress). Other notable threads included **Taylor Blau’s RFC to combine `git repack --geometric` and `--cruft`**, **Junio’s "What’s cooking" report** (marking the start of deep freeze for Git 2.55-rc2), and **ongoing discussions about macOS CI hangs** (with a consensus forming around increasing Apache’s timeout).

---

## Notable threads

### `git replay --linearize` reaches v5, addressing all prior feedback
**Subject:** [PATCH v5 0/3] replay: introduce --linearize option

Toon Claes’s series to add `--linearize` to `git replay` (flattening merge commits to match `git rebase --no-rebase-merges`) reached **v5**, addressing all prior architectural concerns. The most significant change was **reverting the controversial `enum replay_mode` refactoring** (previously patch 1/3) and replacing it with **detailed code comments** explaining the base-selection logic in `pick_regular_commit()`. The series also **fixed a bug in `--onto` handling** when replaying multiple divergent branches and **expanded test coverage** for edge cases.

Junio Hamano’s review of patch 1/3 (the refactoring) was positive, with only a minor style nit about `BUG()` messages. The **interface design question**—whether `--linearize` should mirror `git rebase`’s three-mode design now or later—remains open, but Toon has outlined a possible future evolution (default: preserve topology, `--linearize`: flatten, `--no-merges`: abort, `--replay-cousins`: rebase cousins). The series is now **technically complete** and ready for final review, with no unresolved objections.

---

### Patrick Steinhardt’s ref backend refactoring merged to `next`
**Subject:** [PATCH v6 00/11] refs: avoid chdir_notify_reparent() and fix memory leaks

Patrick Steinhardt’s **11-patch refactoring series** to resolve recursive initialization issues in Git’s reference backend (caused by `includeif.onbranch` conditions) was **merged to `next`** under the new topic name `ps/refs-onbranch-fixes`. The series replaces an earlier workaround with a **lazy-loading design** that defers write-config parsing until the first write operation, eliminating early config reads that could trigger recursion.

Key outcomes:
- **Deferred write-config parsing** (e.g., `core.logAllRefUpdates`, `reftable.blockSize`) via new backend-specific APIs.
- **Recursion guard** in `get_main_ref_store()` to prevent re-entrant initialization.
- **Memory leak fixes** in `chdir_notify` and `repo_clear()`.
- **Test coverage** for `includeif.onbranch` edge cases.

The series was **approved by Jeff King (Peff)** and **reviewed by Justin Tobler**, with Junio Hamano explicitly marking it as ready for `next`. This is a **foundational shift** in how ref backends interact with config, with implications for future backend work (e.g., ODB-based ref stores).

---

### Critical regression in merge-base optimization blocks v3
**Subject:** [PATCH v3 0/8] commit-reach: optimize `paint_down_to_common()` for one-sided histories

Tian Yuchen’s **v3 series** to optimize merge-base calculations for one-sided histories (yielding 100-1000x speedups) was **blocked by a critical regression** in patch 7/8. The patch widened a generation-monotonicity BUG assertion to fire unconditionally, breaking correctness when `min_generation` is not set. Junio Hamano **ejected the series from `seen`** after the test suite caught the failure in `t6600-test-reach.sh`.

The regression is isolated to the BUG assertion and does not affect the rest of the series (patches 1–6/8 and 8/8), which add **trace2 instrumentation**, **documentation**, and **edge-case tests**. The author plans to rework the logic for v4, either by reverting the problematic change or unifying halt conditions while preserving correctness. The series remains **highly anticipated** for its performance impact on asymmetric histories (e.g., repositories with import grafts).

---

### Taylor Blau’s RFC to combine `--geometric` and `--cruft` repacks
**Subject:** [RFC PATCH 00/10] repack: combine --geometric and --cruft

Taylor Blau proposed a **10-patch RFC** to combine `git repack --geometric` and `--cruft`, which are currently mutually exclusive. The series introduces a new `--stdin-packs=follow-reachable` mode to `git pack-objects`, ensuring only reachable objects from rolled-up packs appear in the geometric pack, while unreachable objects are collected into a cruft pack. The implementation includes:
- **Refactoring** of the cruft pack machinery (patches 1–5).
- **New `--stdin-packs=follow-reachable` mode** (patch 8), which walks from reference tips to include only reachable objects from selected packs.
- **Refs snapshot synchronization** (patch 9) to ensure consistency between `pack-objects` and the MIDX bitmap writer.
- **Final integration** (patch 10), teaching `git repack` to relax the `ALL_INTO_ONE` implication of `--cruft` when `--geometric` is given.

Junio Hamano’s review of patch 8/10 identified a **potential correctness issue** in the two-phase traversal logic, where unreachable tags could be incorrectly included in the output pack. The series is **well-motivated** but complex, and its timing (submitted during the -rc phase) suggests it may require additional iterations before merging.

---

### macOS CI hangs: consensus on increasing Apache’s timeout
**Subject:** macOS CI hang in t5551/t5559 – root cause and fix

The thread investigating macOS CI hangs in `t5551` and `t5559` (caused by Apache’s 300-second timeout during `ls-refs` advertisements) reached a **consensus on increasing Apache’s `Timeout` directive** as the most pragmatic solution. Michael Montalbo confirmed that a 5–10 minute timeout resolves the issue in practice, and Jeff King (Peff) endorsed the approach as a "fine mitigation" while noting that ref-packing (already implemented) further reduces timeout risk.

The discussion ruled out client-side mitigations (e.g., `http.lowSpeedLimit`) and upstream Apache fixes as impractical. The next step is to **implement the timeout adjustment** in `t/lib-httpd.sh`. This is a **low-risk, high-impact change** that should eliminate the flakiness without masking underlying issues.

---

## In brief

**`git cat-file --batch-command` security series nears completion**
Pablo Sabater’s GSoC series to add remote object metadata queries to `git cat-file --batch-command` addressed Junio Hamano’s feedback on the `strtoul_szt()` helper, switching to `uintmax_t` to handle platforms where `unsigned long` is smaller than `size_t`. The series is now **ready for merging**, with only a commit message update pending.

**`git history squash` v6 expected to address edge cases**
Harald Nordgren’s `git history squash` series (collapsing commit ranges into a single commit) received **substantive feedback from Phillip Wood** about input validation (e.g., rejecting single-commit ranges) and handling of `fixup!`/`squash!` commits. Harald plans to address these in v6, which will include **stricter validation** and **expanded test coverage**.

**Reftable hardening series adds fuzzing infrastructure**
Patrick Steinhardt’s **11-patch series** to harden the reftable backend against corrupted files added **libFuzzer support** and fixed vulnerabilities (OOB reads/writes, NULL pointer dereferences, uninitialized memory usage). Christian Couder’s review of patch 5/11 suggested **test code refactoring**, but the series is otherwise **ready for review**.

**Libification effort advances with `excludes_file` migration**
Tian Yuchen’s series to move `excludes_file` into `struct repo_config_values` (part of the `the_repository` removal effort) received **mixed feedback**. Junio Hamano approved the current implementation but questioned the **temporary guard** (`if (repo != the_repository)`) to avoid BUG() assertions in uninitialized submodules, suggesting it might mask bugs. SZEDER Gábor identified a **build-breaking issue** in the first patch (unused `repo` parameter), which Junio proposed fixing with an `UNUSED` annotation.

**Rustification: loose object map validation**
Feng Wu’s patch to validate hash algorithms in `ObjectMap::insert()` (part of the Rustification effort) is a **correctness fix** that prevents inconsistent state when invalid ref value types are passed. The patch is **well-tested** and uncontroversial.

**`git history` bugfix for file stream leaks**
Junio Hamano’s patch to fix a file stream leak in `git history reword` (causing issues on Windows) is a **straightforward correctness fix** that consolidates file handling and explicitly closes streams before launching the editor. The patch is **tested and merge-ready**.

---

## On the radar

- **`ps/odb-generalize-prepare`** (Patrick Steinhardt): Needs review. Generalizes the `reprepare()` callback to enable `git grep` with pluggable ODB backends.
- **`ps/reftable-hardening`** (Patrick Steinhardt): Needs review. Hardens the reftable backend against corrupted files and adds fuzzing infrastructure.
- **`hn/branch-delete-merged`** (Harald Nordgren): Needs review. Adds `git branch --delete-merged <branch>` to remove local branches already merged to their remote-tracking branch.
- **`tb/midx-incremental-custom-base`** (Taylor Blau): Needs review. Fixes reachability closure for bitmaps in `git multi-pack-index write --incremental --base`.