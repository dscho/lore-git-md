Here’s the daily digest for **June 26, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A busy day on the Git mailing list (86 emails, 17 threads) saw **security hardening, refactoring, and feature work** dominate the discussion. The **`git cat-file --batch-command` remote object metadata series** neared completion, while **`git replay --linearize`** and **`git history squash`** advanced with architectural debates. A **critical regression** in the `paint_down_to_common()` optimization was caught by the test suite, and **reftable security fixes** landed alongside new fuzzing infrastructure. Junio’s "What’s cooking" report signaled deep freeze for Git 2.55-rc2, with only bugfixes and docs graduating to `master`.

---

### **Notable threads**

#### **`git cat-file --batch-command` remote object metadata (GSoC v14)**
The **13-patch series** implementing `git cat-file --batch-command` for remote object metadata queries (e.g., object size) saw its final review rounds. Pablo Sabater addressed Junio’s feedback on the `strtoul_szt()` helper, switching to `uintmax_t` to handle platforms where `size_t` and `unsigned long` differ. Karthik Nayak and Chandra Pratap flagged minor documentation nits in patches 5–7, all of which were acknowledged. The series is **technically complete**, with only a pending commit message update for `strtoul_szt()`. Junio’s earlier "What’s cooking" report lists it as **waiting for author response**, but today’s traffic suggests it’s nearly ready for `next`.

**Why it matters**: This feature enables efficient remote object queries without full downloads, a key step toward server-side Git operations. The security-hardened design (dynamic placeholder validation, strict protocol checks) sets a high bar for future protocol extensions.

---

#### **`git replay --linearize` (v5)**
Toon Claes posted **v5** of the `--linearize` option for `git replay`, addressing the **boolean vs. enum debate** from v4. The controversial `enum replay_mode` refactoring was reverted in favor of **detailed code comments** explaining the base-commit selection logic in `pick_regular_commit()`. A **bug in `--onto` handling** (incorrect `replayed_base` mapping) was fixed, and test coverage expanded. Junio approved the uncontroversial refactoring patches (1–2/3) but noted a **behavioral difference** with `git rebase --no-rebase-merges` when flattening divergent merges (e.g., `A->X` and `A->Y` merging at `Z`): `git replay --linearize` drops one branch entirely, while `git rebase` rewrites both. Toon acknowledged the discrepancy and proposed documenting it.

**Why it matters**: The series is **ready for `next`** once the behavioral note is added. The `--linearize` option brings server-side rebase functionality closer to parity with `git rebase`, but the interface design debate (whether to mirror all `git rebase` modes now) remains open.

---

#### **`git history squash` (v6 prep)**
Harald Nordgren’s **`git history squash`** series (collapsing commit ranges into a single commit) faced **substantive review** from Phillip Wood. Key concerns:
1. **Input validation**: The command accepts single-commit ranges (e.g., `@^!`) and non-ancestor ranges (e.g., `origin/seen^2^!` from `master`) without warning.
2. **Merge commit handling**: Behavior with external parents (e.g., merges with parents outside the range) is undocumented.
3. **Ref handling**: Clarity on what happens to branches pointing inside the squashed range.

Harald agreed to tighten validation and document edge cases but pushed back on supporting two separate arguments (e.g., `git history squash ^:/base :/tip`), arguing the `<base>..<tip>` form is sufficient. Junio weighed in, demonstrating how `HEAD^{/pattern}` syntax could enable more complex queries, but no consensus emerged.

**Why it matters**: The series is **code-complete** but needs **stricter validation and documentation** before merging. The discussion highlights Git’s tension between flexibility and usability in new commands.

---

#### **`paint_down_to_common()` optimization (v3, regression)**
Kristofer Karlsson’s **v3 series** optimizing `paint_down_to_common()` for one-sided histories hit a **critical regression**: an unconditional BUG assertion in patch 7/8 broke correctness when `min_generation` is unset. Junio ejected the series from `seen` after `t6600-test-reach.sh` failed. Kristofer acknowledged the bug (a human error in unifying halt conditions) and plans a **v4 rework**. Derrick Stolee praised the **trace2 instrumentation** (patch 4/8) and **deterministic step-count assertions**, calling them "clean" and a strong regression guard.

**Why it matters**: The regression underscores the value of **self-verifying tests** in performance patches. The series’ core optimization (100–1000x speedups for asymmetric histories) remains sound, but the fix must preserve correctness across all commit-graph modes.

---

#### **Reftable security hardening (v1)**
Patrick Steinhardt’s **11-patch series** hardened the reftable backend against corrupted files, fixing OOB reads/writes, NULL pointer dereferences, and `abort()` calls. The series also introduced **libFuzzer-based fuzzing infrastructure** (Meson build support, CI integration) to prevent regressions. Christian Couder suggested a **test helper refactor** (patch 5/11) to reduce duplication, but the fixes themselves are uncontroversial. Junio’s "What’s cooking" report lists the topic as **needing review**.

**Why it matters**: This is the first time Git’s build system has been extended for **coverage-guided fuzzing**, a major step for security. The fixes address real vulnerabilities (e.g., zlib OOB writes during log block reconstruction) and are critical for the reftable backend’s stability.

---

#### **ODB abstraction and libification**
Two series advanced Patrick Steinhardt’s **ODB abstraction effort**:
1. **`ps/odb-generalize-prepare` (v1)**: Generalized the `reprepare()` callback into a `prepare()` callback with an optional `ODB_PREPARE_FLUSH_CACHES` flag, enabling `git grep` to work with pluggable ODBs. Toon Claes raised **design questions** about downcasting and flag usage, but the series is otherwise ready for review.
2. **`ps/connected-generic-promisor-checks` (v3)**: Refactored connectivity checks to use generic ODB iteration instead of packfile internals. Christian Couder approved the final patch, and Junio’s "What’s cooking" report lists it as **ready for `next`**.

**Why it matters**: These series are **foundational** for pluggable ODB backends (e.g., reftable, custom storage). The `ps/connected-generic-promisor-checks` series is particularly important for repositories that don’t use packfiles.

---

### **In brief**
- **`git repack --geometric` + `--cruft` (RFC)**: Taylor Blau proposed a **10-patch series** combining the two modes, introducing `--stdin-packs=follow-reachable` to `git pack-objects`. The series is **complex** (reachability traversal, refs snapshots) and submitted during the -rc phase, so it may require multiple iterations.
- **`git history` bugfix**: Junio fixed a **file stream leak** in `fill_commit_message()` that could block editors on Windows.
- **Libification**: Tian Yuchen’s series moving `excludes_file` into `struct repo_config_values` faced **build-breaking feedback** (unused parameter) and a **design debate** about submodule guards. Junio suggested marking the parameter `UNUSED` as a temporary fix.
- **Rustification**: A patch fixed a **correctness issue** in `ObjectMap::insert()` by validating hash algorithms.
- **CI hangs**: The macOS CI hang in `t5551`/`t5559` was traced to an **Apache `mod_http2` bug**. The consensus is to **increase Apache’s timeout** as a workaround.

---

### **On the radar**
- **`ps/reftable-hardening`**: The reftable security fixes need review, but the fuzzing infrastructure is a **long-term win** for Git’s security posture.
- **`jt/receive-pack-use-odb-transactions`**: A 6-patch series refactoring `git-receive-pack` to use ODB transactions (instead of `tmp_objdir`) is **waiting for author response**.
- **`hn/branch-push-slip-advice`**: A 3-patch series adding advice for typos like `git push origin/main` (suggesting `origin main`) is **waiting for author response**.
- **`tc/replay-linearize`**: The interface design debate (whether to mirror all `git rebase` modes) may resurface in future iterations.