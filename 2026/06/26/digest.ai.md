# The Git Project Mailing List Daily Digest

**2026/06/26**

**The day in brief**
A busy day on the Git mailing list with 86 emails across 17 threads. The standout developments: a security-hardened `git cat-file --batch-command` series reaches final readiness; Patrick Steinhardt's ref backend refactoring lands in `next`; and a critical regression surfaces in a performance optimization for merge-base calculations. The day also saw progress on ODB abstraction, Rustification, and repack machinery, alongside routine translation updates and CI fixes.

---

## Notable threads

### **`git cat-file --batch-command` remote object info reaches final readiness**
Pablo Sabater's GSoC project to add remote object metadata queries to `git cat-file --batch-command` is now complete at v14, with all substantive feedback addressed. The series implements a security-hardened protocol v2 extension allowing clients to request object metadata (currently size) from remotes without downloading full objects. Dynamic format placeholder validation adapts to server capabilities, and comprehensive memory safety improvements were added post-v14. Junio Hamano and Karthik Nayak identified minor documentation nits in the `strtoul_szt()` helper and function relocation commits, which Pablo acknowledged and will correct. The series demonstrates cross-platform correctness and is ready for merging, pending these final documentation tweaks.

---

### **Ref backend refactoring lands in `next`**
Patrick Steinhardt's 11-patch series modernizing Git's reference backend infrastructure has been merged into Junio Hamano's `next` branch as topic `ps/refs-onbranch-fixes`. The series resolves recursive initialization issues triggered by `includeif.onbranch` conditions through a lazy-loading design that defers write-config parsing until the first write operation. Jeff King (Peff) endorsed the architectural approach, calling it "good" and "not too painful to maintain," while Justin Tobler gave final approval after Patrick addressed minor feedback. The changes span 34 files and introduce deferred write-config parsing, dynamic write option passing, and a recursion guard. This foundational work enables future backend modularity and addresses long-standing initialization-order constraints.

---

### **Critical regression in merge-base optimization**
A performance optimization series for `paint_down_to_common()` hit a snag when Junio C Hamano identified a critical regression in patch 7/8. The patch, which implements early termination when one side of a merge-base query exhausts its commit queue, incorrectly widened a generation-monotonicity BUG assertion to fire unconditionally, breaking correctness when `min_generation` is not set. The regression caused test failures in `t6600-test-reach.sh` (test 12, `get_merge_bases_many`), where the assertion fired instead of returning expected merge bases. Kristofer Karlsson and the author (Tian Yuchen) acknowledged the bug and plan to rework the logic for v4, either by reverting the problematic change or unifying halt conditions while preserving correctness. The rest of the series—including edge-case tests, trace2 instrumentation, and documentation—remains unaffected and ready for review.

---

### **ODB abstraction advances with promisor object handling**
Patrick Steinhardt's series generalizing promisor object handling in connectivity checks has been fully reviewed and approved. The 4-patch series refactors Git's connectivity logic to remove hardcoded packfile assumptions, enabling ODB backend independence. Christian Couder and Junio Hamano gave final approval to the fourth patch, which replaces `find_pack_entry_one()` with the generic `odb_for_each_object_ext()` API. This work is critical for Patrick's broader ODB abstraction effort and paves the way for pluggable storage backends. The changes are confined to `connected.c` and the test suite, with no user-facing behavior changes.

---

### **Rustification: hash algorithm validation in ObjectMap**
A standalone patch from the Rustification effort adds validation for hash algorithms in `ObjectMap::insert()`. The patch enforces the invariant that object IDs passed to `insert()` must use expected hash algorithms (one for storage, one for compatibility), returning a new `ObjectMapInsertError` enum for mismatched or unknown algorithms. The change is narrowly scoped to `src/loose.rs` and includes expanded test coverage. This correctness fix addresses a previously unchecked assumption in the loose object map, a core Rust data structure used for object mapping.

---

## In brief

**Reftable security hardening** -- Patrick Steinhardt's 11-patch series hardens the reftable backend against corrupted files, fixing OOB reads/writes, NULL pointer dereferences, and aborts during parsing. Christian Couder suggested a minor test refactoring to reduce duplication.

**`git history squash` nears completion** -- Harald Nordgren's series to add `git history squash` for folding commit ranges received substantive feedback from Phillip Wood on edge cases (single-commit ranges, non-ancestor ranges, `fixup!`/`squash!` handling). Harald plans to address these in v6.

**Git 2.55.0 translation cycle closes** -- Junio Hamano reminded translators that the submission deadline for Git 2.55.0 is June 27, with the release scheduled for June 29.

**CI fixes for macOS hangs** -- Michael Montalbo and Jeff King (Peff) reached consensus on increasing Apache's `Timeout` directive to mitigate CI hangs in `t5551` and `t5559` caused by Apache bug 70131. The fix avoids client-side mitigations and directly addresses the root cause.

**`git repack` RFC combines geometric and cruft modes** -- Taylor Blau's 10-patch RFC proposes combining `--geometric` and `--cruft` repack modes, introducing `--stdin-packs=follow-reachable` and `--refs-snapshot` for precise reachability filtering. Junio identified a correctness issue in the two-phase traversal logic that could retain unreachable objects.

**`excludes_file` migration into `repo_config_values`** -- Tian Yuchen's series to move the global `excludes_file` variable into `struct repo_config_values` received critical feedback from Junio Hamano on a temporary guard and from SZEDER Gábor on a build-breaking unused parameter. Junio proposed a pragmatic fix using the `UNUSED` macro.

**`git history` file stream leak fixed** -- Junio Hamano submitted a merge-ready patch fixing an inefficient file-handling pattern and stream leak in `git history --reword`, improving Windows compatibility.

**Build system quieting for gitk and git-gui** -- Harald Nordgren and Johannes Sixt discussed integration timing for patches that align gitk and git-gui's translation catalog generation with core Git's quiet build conventions.

**`git replay --linearize` fully merged** -- Toon Claes's series introducing `--linearize` to `git replay` was merged, with Junio Hamano identifying a post-merge behavioral edge case: merge commit divergence handling differs from traditional flattening rebase.

## On the radar

**`git repack` geometric+cruft integration** -- Taylor Blau's RFC to combine `--geometric` and `--cruft` repack modes requires a fix for the reachability filtering correctness issue identified by Junio Hamano. The series is complex and submitted during the -rc phase, so additional iterations are likely.

**Merge-base optimization regression** -- The critical regression in Tian Yuchen's `paint_down_to_common()` optimization series blocks merging until the generation-monotonicity BUG assertion is fixed. The author plans a v4 rework.

**`excludes_file` migration** -- Tian Yuchen's series to move `excludes_file` into `struct repo_config_values` needs a v3 to address the build-breaking unused parameter and Junio's concerns about the temporary guard.