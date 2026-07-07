Here’s the daily digest for **June 26, 2026**, covering the Git mailing list’s key developments:

---

### **The day in brief**
A busy day with **86 emails across 17 threads**, dominated by **performance optimizations, security hardening, and refactoring**—but also marked by a **critical regression** in a merged series. The `paint_down_to_common()` optimization (now ejected from `seen`) and Patrick Steinhardt’s **reftable security hardening** series led the technical discussions, while Taylor Blau’s **RFC to combine `--geometric` and `--cruft` repacks** introduced a major workflow change. Junio’s "What’s cooking" report set the stage for Git 2.55-rc2’s deep freeze, with only bugfixes and docs graduating to `master`.

---

### **Notable threads**

#### **1. `paint_down_to_common()` optimization: Regression blocks progress**
**Headline**: *Merge-base speedup ejected from `seen` after critical test failure*
The v3 series to optimize `paint_down_to_common()` for one-sided histories (yielding **100-1000x speedups**) hit a snag when a **widened BUG assertion** broke correctness for callers not setting `min_generation`. Junio ejected the series from `seen` after `t6600-test-reach.sh` failed, confirming the test suite’s robustness. Kristofer Karlsson (author) acknowledged the regression and plans a v4 rework, while Derrick Stolee praised the **trace2 instrumentation** (patch 4/8) as "clean" and "over-achieving." The core optimization remains sound, but the regression underscores the fragility of generation-number invariants in merge-base calculations.
**Status**: *Blocked on regression fix; v4 expected soon.*

---

#### **2. Reftable security hardening: Fuzzing uncovers critical bugs**
**Headline**: *Patrick Steinhardt’s 11-patch series hardens reftable against corrupted files*
This **security-focused series** introduces **libFuzzer-based fuzzing** to Git’s build system (Meson support) and fixes **OOB reads/writes, NULL dereferences, and `abort()` calls** in the reftable backend. Highlights:
- **Patch 5/11**: Fixes an OOB write in log block reconstruction (zlib underflow).
- **Patch 11/11**: Validates file truncation to prevent OOB reads in footer parsing.
- **Fuzzing infrastructure**: New `oss-fuzz/fuzz-reftable.c` target runs in CI via `ci/run-build-and-minimal-fuzzers.sh`.
Christian Couder reviewed patch 5/11, suggesting test helper extraction (non-blocking). The series is **ready for review** and critical for reftable’s stability.
**Status**: *Needs review; no objections yet.*

---

#### **3. `git repack --geometric` + `--cruft`: Taylor Blau’s RFC**
**Headline**: *Combining repack modes to streamline large-repo maintenance*
Taylor Blau’s **10-patch RFC** proposes merging `--geometric` and `--cruft` repack modes, which are currently mutually exclusive. Key innovations:
- **`--stdin-packs=follow-reachable`**: A new `git pack-objects` mode that includes only reachable objects from selected packs (critical for geometric rollups).
- **Refs snapshot**: Ensures consistency between `pack-objects` and MIDX bitmap writers.
- **Cruft pack logic**: Respects geometric split points to exclude packs above the split from cruft collection.
The series is **well-tested** (8 new tests in `t/t7704-repack-cruft.sh`) but submitted during the **2.55-rc2 freeze**, so it’s likely to iterate post-release. Junio flagged a **potential correctness issue** in patch 8/10 (unreachable tags being retained), which Taylor will need to address.
**Status**: *RFC; expecting feedback post-2.55.*

---

#### **4. `git history squash`: Edge cases delay final review**
**Headline**: *Phillip Wood’s feedback reveals input validation gaps*
Harald Nordgren’s `git history squash` (v5) to fold commit ranges into a single commit faces **usability concerns**:
- **Single-commit ranges** (e.g., `@^!`) are accepted but meaningless.
- **Non-ancestor ranges** (e.g., `origin/seen^2^!` from `master`) pass silently.
- **Merge commits with external parents** lack documented behavior.
Phillip Wood’s review prompted a v6 plan to tighten validation, while Junio demonstrated alternative syntaxes (e.g., `HEAD^{/pattern}..:/tip`) to handle complex cases. The core three-way merge logic remains uncontested.
**Status**: *Expecting v6 with stricter validation.*

---

#### **5. ODB abstraction: `reprepare()` → `prepare()` refactoring**
**Headline**: *Patrick Steinhardt’s 2-patch series generalizes ODB callbacks*
This **refactoring** replaces the `reprepare()` callback with a more flexible `prepare()` that accepts `enum odb_prepare_flags` (`ODB_PREPARE_FLUSH_CACHES`). The change enables `git grep` to work with pluggable ODB backends by decoupling cache invalidation from preparation. Toon Claes raised **design questions** about downcasting and flag usage in patch 1/2, but the series is otherwise **ready for review**.
**Status**: *Needs review; part of ODB abstraction effort.*

---

### **In brief**
- **`git cat-file --batch-command`**: Pablo Sabater addressed Junio’s feedback on `strtoul_szt()` (patch 2/13), switching to `uintmax_t` for cross-platform safety. The **security-hardened series** (v14) is now complete.
- **`git replay --linearize`**: Toon Claes posted **v5** with improved comments and a bug fix for `--onto` handling. The **boolean refactoring debate** is resolved, but Junio noted a behavioral difference from `git rebase --no-rebase-merges`.
- **Ref backend fixes**: Patrick Steinhardt’s **v6 series** (lazy-loading for `includeif.onbranch`) was **merged to `next`** as `ps/refs-onbranch-fixes`.
- **Libification**: Tian Yuchen’s patch to move `excludes_file` into `struct repo_config_values` hit a **build-breaking issue** (unused `repo` parameter) and a **design debate** over submodule guards. Junio suggested `UNUSED` annotations as a temporary fix.
- **Rustification**: Feng Wu fixed a **correctness bug** in `ObjectMap::insert()` (Rust loose object map), validating hash algorithms before insertion.
- **CI hangs**: Michael Montalbo identified an **Apache `mod_http2` bug** (upstream 70131) as the root cause of macOS CI hangs in `t5551`/`t5559`. The consensus leans toward **increasing Apache’s `Timeout`** as a pragmatic fix.

---

### **On the radar**
- **`ps/odb-drop-whence`**: Awaiting review; removes the `whence` field from `struct object_info` in favor of opt-in backend info.
- **`jt/receive-pack-use-odb-transactions`**: Refactors `git-receive-pack` to use ODB transactions (backend-agnostic).
- **`hn/branch-push-slip-advice`**: Adds typo-fix advice for `git push origin/main` vs. `git branch origin main`.

---

### **Editorial note**
Today’s traffic reflected Git’s **dual focus** on **performance** (merge-base optimizations, repack workflows) and **security** (reftable hardening, fuzzing). The `paint_down_to_common()` regression serves as a reminder that **even well-tested optimizations can break invariants**, while Taylor Blau’s RFC highlights the **growing complexity of repack strategies** in large repositories. Junio’s "What’s cooking" report reinforced the **2.55-rc2 freeze**, with only bugfixes and docs graduating to `master`. Expect follow-ups on the regression and RFCs post-release.