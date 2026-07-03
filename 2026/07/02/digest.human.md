Here is the digest for **2026/07/02**, covering the Git mailing list traffic in a concise, front-page style:

---

## The day in brief
**2026/07/02** was a **heavy but focused** day on the Git mailing list, with **68 emails across 21 threads**. The standout themes were **post-merge bugfixes** (notably for `git replay --linearize` and `git history squash`), **ODB/refactoring work** (Patrick Steinhardt’s `struct object_info` cleanup), and **memory-leak fixes** (Jeff King’s hash-implementation series). The **Git v2.55.0 release** also drew lighthearted attention for its contributor list quirk. If you only read one thing today, make it the **`git replay --linearize` regression fix**—it’s urgent and ready for re-merge.

---

## Notable threads

### **`git replay --linearize` regression fix (v6)**
**Toon Claes** posted a **critical v6 series** to fix a **silent commit-dropping regression** in the recently merged `git replay --linearize` feature. The bug, introduced in v5, caused only the tip commit to be replayed in single-branch scenarios due to a broken `replayed_base` mechanism. The series also **documents edge cases** (e.g., divergent merges) and **expands test coverage** for root commits, multiple branches, and `--contained` interactions. **Johannes Schindelin** and **Patrick Steinhardt** drove the design discussions, with Schindelin advocating for UX clarity over consistency with `git rebase`. The series is **ready for re-merge** to replace the flawed v5, with no further design changes expected.

**Key takeaway**: If you use `git replay`, this fix is **urgent**—v5 silently drops commits. The v6 series is a **bugfix/documentation update**, not a new feature.

---

### **`git history squash` recoverability and reflog critique**
The **`git history squash`** series (Harald Nordgren) saw **substantive discussion** about **recoverability gaps** in Git’s design. **Matt Hunter** and **Phillip Wood** highlighted that `git reset --hard` is insufficient to undo operations where `--update-refs` moves multiple branches, as Git’s reflog lacks visibility into which refs were affected. **Patrick Steinhardt** proposed an **oplog (operations log)** to enable atomic undo of multi-ref operations, citing Jujutsu (`jj`) and git-branchless as precedents. **Junio C Hamano** clarified that the reflog was never designed for undo, pushing back on overloading it. The series itself is **code-complete (v6)** and ready for final review, but the discussion exposes a **systemic limitation** in Git’s recovery tooling.

**Key takeaway**: The `squash` feature is technically sound, but its `--update-refs` behavior reveals a **broader UX gap**—Git lacks a way to atomically undo complex operations. The oplog idea is a **long-term architectural proposal**, not a blocker for this series.

---

### **ODB refactoring: `struct object_info` cleanup (v2)**
**Patrick Steinhardt** posted **v2 of his 6-patch series** refactoring `struct object_info` to replace the coarse `whence` field with a backend-specific `struct odb_source_info`. The series is **conceptually approved** by Junio, with only a minor reroll needed to rename `sourcep` to `source_infop` (per Justin Tobler’s feedback). The changes are **mechanical but foundational**, touching 14 files across the ODB, packfile, and builtin subsystems. The goal is to enable **multi-source object resolution** and **pluggable ODB backends**, part of Steinhardt’s long-running ODB abstraction effort.

**Key takeaway**: This is a **critical intermediate step** for ODB modularity. The series is **ready for `next`** after the minor rename, with no open technical questions.

---

### **Memory-leak fixes in non-default hash implementations (v1)**
**Jeff King** submitted a **9-patch series** plugging memory leaks in Git’s hash implementations when using non-default backends (OpenSSL SHA-256, libgcrypt). The leaks, discovered via `SANITIZE=leak`, affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. The series introduces a new `git_hash_discard()` primitive and systematically applies it to leaky code paths. **Junio approved patch 1/9** (removing a redundant `discard_hashfile()` function), and the rest of the series is **unreviewed but uncontroversial**. The fixes are **narrowly scoped** to non-default configurations, so impact is limited.

**Key takeaway**: If you use OpenSSL or libgcrypt with Git, these patches are **important correctness fixes**. The series is **well-structured** and likely to merge as-is.

---

### **`GIT_TEST_LONG` tests: reliability and efficiency (v1)**
**Toon Claes** posted a **9-patch series** to make `GIT_TEST_LONG` tests reliable and efficient enough to run in CI. The series includes **test correctness fixes** (skipping broken tests on 32/64-bit systems), **efficiency improvements** (reducing disk usage and runtime), and **CI configuration changes** (enabling `GIT_TEST_LONG` in GitLab CI). **SZEDER Gábor** raised a **policy question** about patch 3/9: whether the `EXPENSIVE` prerequisite should be retained for a test that still uses >1 GiB of memory, even though its runtime is now negligible. **Jeff King** endorsed the series’ efficiency improvements but noted that the `test_copy_bytes` helper (used in patch 3) might be ripe for modernization.

**Key takeaway**: This series **unblocks CI coverage** for long-running tests. The `EXPENSIVE` debate is a **policy question** (runtime vs. resource usage) rather than a technical blocker.

---

## In brief
> **`git rev-list --exclude-first-parent-only` bug** -- Michael Hore reported a logic flaw in `process_parents()` where explicitly specified commits cause `--exclude-first-parent-only` to incorrectly exclude commits. No patch yet; diagnosis points to `revision.c`.

> **`git rm -n *.json` recursion bug** -- Евгений Плискин reported that `git rm -n *.json` recursively removes JSON files from subdirectories, contradicting the documentation. Likely a shell glob vs. Git path-matching issue.

> **`git apply` memory leak and state corruption** -- A standalone patch fixes a leak in `find_header()` where abandoned Git-style diff headers corrupt subsequent parsing. Includes regression tests.

> **Reftable hardening (v2)** -- Patrick Steinhardt’s 12-patch series hardening the reftable backend against corruption is **technically complete**, with all feedback addressed. Includes fuzzing infrastructure and Meson libFuzzer support.

> **Git v2.55.0 released** -- Junio announced the release, drawing lighthearted attention for including "Claude Sonnet 4.6" in the contributor list. No technical follow-up.

> **Git for Windows 2.55.0(2) hotfix** -- Johannes Schindelin re-enabled NTLM authentication as an opt-in deprecated feature, addressing a premature disabling in 2.55.0. Also marks the final release to support Windows 8.1.

> **Test modernization** -- A patch modernized `t9811-git-p4-label-import.sh` to use `test_path_is_file`/`test_path_is_missing` instead of legacy shell constructs.

---

## On the radar
- **`git replay --linearize` v6** is **ready for re-merge** to fix the silent commit-dropping regression. Track this if you use `git replay`.
- **ODB refactoring (`ps/odb-drop-whence`)** is **ready for `next`** after a minor reroll. Critical for future ODB modularity.
- **Memory-leak fixes in hash implementations** are **unreviewed but uncontroversial**. Important for OpenSSL/libgcrypt users.
- **`GIT_TEST_LONG` CI coverage** is expanding to GitLab, but the `EXPENSIVE` debate may influence future test labeling policy.