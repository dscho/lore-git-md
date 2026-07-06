Here’s the daily digest for **June 26, 2026**, covering the Git mailing list’s activity:

---

### **The day in brief**
A busy day with **86 emails across 17 threads**, dominated by **performance optimizations**, **security hardening**, and **refactoring efforts**. The standout developments:
1. **`git replay --linearize` v5** landed, resolving architectural debates and fixing edge cases.
2. **Reftable security hardening** (11 patches) introduced fuzzing infrastructure and fixes for corrupted-file vulnerabilities.
3. **`paint_down_to_common()` optimization** hit a regression but remains on track for dramatic speedups.
4. **`git repack --geometric` + `--cruft` RFC** proposed unifying two previously incompatible modes.

---

### **Notable threads**

#### **`git cat-file --batch-command` remote object metadata (GSoC v14)**
**Headline**: Final nits addressed; series ready to merge.
Pablo Sabater’s **security-hardened** series for querying remote object metadata (e.g., size) via `git cat-file --batch-command` is now **technically complete**. Today’s traffic focused on **minor documentation fixes** (e.g., `strtoul_szt()` return type, function naming consistency) and **commit message clarifications**. Junio’s earlier "Will merge to 'next'" signal still stands, with no new objections. The series adds **dynamic format placeholder validation** (adapting to server capabilities) and **memory safety improvements**, making it a critical step toward server-side Git operations.
*Key detail*: The `strtoul_szt()` helper was revised to use `uintmax_t` for cross-platform safety, addressing Junio’s concern about type size mismatches.

---

#### **`git replay --linearize` v5**
**Headline**: Architectural debates resolved; series merged.
Toon Claes’s **`--linearize` option for `git replay`** (flattening merge commits to match `git rebase --no-rebase-merges`) reached **v5**, addressing all prior feedback:
- **Boolean refactoring reverted**: Replaced with detailed code comments to improve readability (resolving Junio and Patrick Steinhardt’s concerns).
- **Bug fix**: Corrected `--onto` handling for divergent branches.
- **Test coverage**: Added edge cases for root commits and merge topologies.
Junio’s **final review** flagged a behavioral difference with `git rebase` (replay drops one branch entirely when linearizing divergent merges), but this was deemed acceptable with documentation. The series is now **merged into `next`**.
*Key detail*: The interface design debate (whether to mirror `git rebase`’s three modes) remains open for future work, but the current implementation is considered sufficient.

---

#### **Reftable security hardening (11 patches)**
**Headline**: Fuzzing infrastructure added; vulnerabilities fixed.
Patrick Steinhardt’s **security-hardening series** for the reftable backend introduced **libFuzzer integration** and fixed **9 vulnerabilities** (OOB reads/writes, NULL dereferences, `abort()` calls) discovered via fuzzing. Today’s discussion focused on **test hygiene** (Christian Couder suggested extracting a helper to reduce duplication), but the series is **ready for review**. The fuzzer (`oss-fuzz/fuzz-reftable.c`) exercises the parser’s three main entry points and runs in CI.
*Key detail*: Patch 5/11 fixed an OOB write in log block reconstruction by adding a sanity check for block size ≥ header size + 4-byte block header + 2-byte restart counter.

---

#### **`paint_down_to_common()` optimization (v3)**
**Headline**: Regression identified; series blocked.
Tian Yuchen’s **100–1000x speedup** for merge-base queries in asymmetric histories hit a **critical regression** in patch 7/8: an unconditional BUG assertion broke correctness when `min_generation` is unset. Junio ejected the series from `seen`, and the author plans a **v4 rework**. The rest of the series (patches 1–6/8 and 8/8) remains **stable and well-tested**, with **deterministic step-count assertions** for all commit-graph modes.
*Key detail*: The regression was caught by `t6600-test-reach.sh` (test 12), validating the test suite’s robustness.

---

#### **`git repack --geometric` + `--cruft` RFC**
**Headline**: Ambitious unification proposed.
Taylor Blau’s **10-patch RFC** aims to combine `--geometric` and `--cruft` repack modes, which are currently mutually exclusive. The series introduces:
- **`--stdin-packs=follow-reachable`**: A new `git pack-objects` mode to include only reachable objects from selected packs.
- **Refs snapshot**: Ensures consistency between `pack-objects` and MIDX bitmap writers.
- **Geometric split integration**: Teaches the cruft writer to respect the geometric split point.
Junio’s review flagged a **correctness issue** in the two-phase traversal logic (unreachable tags could be incorrectly included), but the series is **well-motivated** for large repositories. Given the timing (submitted during -rc phase), expect **further iterations**.
*Key detail*: The final patch adds 8 new tests covering loose unreachable objects, pack deletion, and `--combine-cruft-below-size`.

---

### **In brief**
- **`git history squash` v6**: Harald Nordgren addressed Phillip Wood’s edge-case feedback (e.g., rejecting single-commit ranges), but Junio’s input on input syntax flexibility may require another iteration.
- **ODB abstraction**: Patrick Steinhardt’s `ps/odb-generalize-prepare` (generalizing `reprepare()`) and `ps/connected-generic-promisor-checks` (generic promisor checks) received **surface-level reviews** from Toon Claes.
- **Libification**: Tian Yuchen’s series moving `excludes_file` into `struct repo_config_values` hit a **build-breaking issue** (unused parameter) and a **design debate** about submodule guards.
- **macOS CI hangs**: Consensus emerged to **increase Apache’s `Timeout`** to 600 seconds, resolving `curl 18` mid-transfer aborts in `t5551`/`t5559`.
- **Rustification**: Feng Wu fixed a correctness issue in `ObjectMap::insert()` (validating hash algorithms).

---

### **On the radar**
- **`ps/refs-onbranch-fixes`**: Patrick Steinhardt’s **lazy-loading ref backend** series (merged into `next`) may graduate to `master` soon.
- **`ps/reftable-hardening`**: The security-hardening series needs **final reviews** before merging.
- **`jt/receive-pack-use-odb-transactions`**: Awaiting author response to Junio’s feedback.