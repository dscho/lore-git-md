Here’s the digest for **July 2, 2026**—a day of post-release cleanup, architectural debates, and targeted bugfixes.

---

### The day in brief
Git development today was a mix of **post-2.55.0 follow-ups** and **long-running architectural discussions**. The `git replay --linearize` series reached a critical milestone with a regression fix, while the `git history squash` thread exposed systemic gaps in Git’s undo capabilities. Memory leaks in hash implementations and test suite flakiness also drew attention. The tone was **pragmatic but forward-looking**, with contributors balancing immediate fixes against broader design questions.

---

### Notable threads

#### **`git replay --linearize` regression fixed (v6)**
Toon Claes’s series to add `--linearize` to `git replay` (flattening merge commits into a linear history) is now **technically complete** after a critical regression fix. The v6 reroll restores the `replayed_base` logic, ensuring all commits in a single-branch replay are properly linearized—addressing Johannes Schindelin’s report of silent commit dropping. The series also clarifies the **intentional design choice** to produce a single linear sequence (even if it duplicates shared history) and documents the behavioral difference from `git rebase --no-rebase-merges`. While the CLI design debate (consistency with `git rebase` vs. UX clarity) remains unresolved, the implementation is now **ready for final review**.

**Why it matters**: This feature is a key piece of Git’s server-side tooling, and the regression fix ensures it behaves predictably. The thread also highlights a **growing tension** between CLI consistency and UX experimentation in new commands.

---

#### **`git history squash` and Git’s undo problem**
The `git history squash` series (Harald Nordgren) is code-complete, but a **systemic critique** of Git’s undo capabilities emerged. Matt Hunter and Phillip Wood noted that `git reset --hard` is insufficient to undo operations like `squash --update-refs`, which move multiple branches. Patrick Steinhardt proposed an **"oplog"** (operations log) to enable atomic undo, while Junio C Hamano clarified that the reflog was never designed for this purpose. The discussion underscores a **missing affordance** in Git’s design: high-level operations lack recoverability metadata.

**Why it matters**: This isn’t just about `squash`—it’s a **broader UX gap** affecting `rebase --update-refs`, `merge`, and other multi-ref commands. The oplog idea could reshape how Git handles undo in the long term.

---

#### **ODB refactoring: `struct object_info` redesign (v2)**
Patrick Steinhardt’s six-patch series to replace `struct object_info`’s `whence` field with a backend-specific `source_infop` pointer is now **conceptually approved** by Junio. The v2 reroll renames `sourcep` to `source_infop` to avoid ambiguity, addressing Justin Tobler’s feedback. The series is a **foundational step** toward pluggable ODB backends, though Junio flagged a readability concern about an implicit upcast (`oi->source_infop->source = &source->base`).

**Why it matters**: This is the latest in Patrick’s **ODB abstraction effort**, which aims to make Git’s object storage more modular. The changes are mechanical but critical for future work.

---

#### **Memory leaks in hash implementations (9-patch series)**
Jeff King (Peff) sent a **nine-patch series** plugging memory leaks in Git’s hash implementations when using non-default backends (OpenSSL, libgcrypt). The leaks were discovered via `SANITIZE=leak` and affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. The series introduces `git_hash_discard()` and systematically applies it across the codebase. Junio approved the first patch, and the rest are likely to follow.

**Why it matters**: While the leaks only affect non-default hash backends, the fixes are **low-risk and high-impact** for users who configure Git to use OpenSSL or libgcrypt. The series also highlights gaps in CI coverage for leak detection.

---

#### **`GIT_TEST_LONG` tests: reliability and efficiency (9-patch series)**
Toon Claes’s series to make `GIT_TEST_LONG` tests reliable and efficient for CI is **nearly complete**. The patches fix broken tests (e.g., skipping 32-bit-only tests on 64-bit systems), improve performance (e.g., replacing a 6-minute `dd` loop with `genzeros`), and enable `GIT_TEST_LONG` in GitLab CI. SZEDER Gábor and Jeff King raised **substantive questions** about whether some tests should retain the `EXPENSIVE` label (e.g., due to high memory usage), but the series is otherwise uncontroversial.

**Why it matters**: This is a **CI infrastructure win**, ensuring that expensive tests run reliably without hanging or consuming excessive resources. The discussion about `EXPENSIVE` vs. `GIT_TEST_LONG` may influence how Git defines "expensive" tests in the future.

---

### In brief
- **`git rev-list --exclude-first-parent-only` bug**: Michael Hore reported unexpected behavior when additional commits are specified on the command line. The issue lies in `process_parents()` in `revision.c`.
- **`git apply` memory leak**: Zephyr Yao fixed a leak in `find_header()` where abandoned Git-style diff headers could corrupt state in `struct patch`.
- **`git rm -n *.json` recursion**: Евгений Плискин reported that `git rm -n` recursively removes files despite documentation claiming it shouldn’t. This appears to be a core Git issue, not platform-specific.
- **Reftable hardening (v2)**: Patrick Steinhardt’s 12-patch series to harden the reftable backend against corruption is now **technically complete**, with all feedback addressed. The series includes fuzzing infrastructure and fixes for out-of-bounds reads/writes.
- **Meson build race**: D. Ben Knoble’s patch to fix a race condition in generating `hook-list.h` was approved by Adrian Ratiu and Patrick Steinhardt. The fix is minimal and ready for merging.
- **Git v2.55.0 released**: Weijie Yuan noted a humorous quirk in the contributor list (the name "Claude Sonnet 4.6" appeared, likely an AI model).
- **Git for Windows 2.55.0(2)**: Johannes Schindelin announced a hotfix re-enabling NTLM authentication, previewing its planned deprecation later in 2026.
- **Test modernization**: Marcelo Machado Lage updated `t9811-git-p4-label-import.sh` to use modern test helpers (`test_path_is_file`/`test_path_is_missing`).

---

### On the radar
- **`git replay --linearize` CLI design**: The debate over `--linearize` vs. `git rebase`’s `--rebase-merges` syntax remains unresolved. Patrick Steinhardt’s demand for **explicit justification in commit messages** for any divergence may set a precedent for future CLI changes.
- **Oplog for atomic undo**: Patrick’s proposal for an operations log to enable atomic undo of multi-ref operations is **visionary but long-term**. It could resurface in future discussions about Git’s UX.
- **`git history` recoverability**: The `squash --update-refs` critique highlights a **systemic gap** in Git’s undo capabilities. Short-term workarounds (e.g., `--dry-run --verbose`) may be needed, but the oplog idea is the most promising long-term solution.