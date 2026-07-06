Here’s the daily digest for **July 2, 2026**, covering the Git mailing list’s activity:

---

### **The day in brief**
A busy day (68 emails, 21 threads) with **critical post-merge fixes** dominating the discussion—particularly for `git replay --linearize` (silent commit dropping) and `git history squash` (recoverability concerns). **Memory leaks** in hash implementations and **CI/test infrastructure** improvements also saw significant progress, while a **Git for Windows hotfix** and **Git Rev News** rounded out the non-technical updates. The tone was collaborative, with maintainers and contributors aligning on fixes and long-term solutions.

---

### **Notable threads**

#### **1. `git replay --linearize` regression fix (v6)**
**Headline:** *Silent commit dropping in `--linearize` resolved; series ready for final review*
**Author:** Toon Claes
**Status:** **v6 posted**, addressing all prior feedback, including Johannes Schindelin’s critical regression report.

The `git replay --linearize` feature (merged in Git 2.55) had a **serious bug**: when replaying a single branch containing merge commits, it silently dropped all intermediate commits, replaying only the tip. The v6 series restores the `replayed_base` logic to ensure all commits are flattened into a single linear topology, even with multiple branches. The commit messages and documentation now clearly explain the behavior (e.g., dropping merges, duplicating shared history) and edge cases (e.g., `--ref` workarounds for branch independence).

**Key changes:**
- **Patch 2/3** refactors base-selection logic to improve clarity.
- **Patch 3/3** implements `--linearize` with thorough test coverage, including divergent-branch scenarios.
- **CLI debate settled**: The series retains `--linearize` (vs. `git rebase`’s `--rebase-merges` syntax), with explicit justification in the commit message.

**Next steps:** Junio’s final review pending; no further design changes expected.

---

#### **2. `git history squash` recoverability and template debate**
**Headline:** *Recoverability gaps exposed; oplog proposed as long-term solution*
**Authors:** Harald Nordgren, Patrick Steinhardt, Phillip Wood
**Status:** **v6 code-complete**, but **systemic recoverability concerns** spark broader discussion.

The `git history squash` series (now v6) is technically ready, but **user feedback** from Matt Hunter and Phillip Wood highlighted a **critical usability gap**: `git reset --hard` is insufficient to undo operations where `--update-refs` moves multiple branches, as Git’s reflog lacks visibility into which refs were affected. Phillip expanded this into a **systemic critique**, noting that Git lacks a way to atomically undo multi-ref operations (unlike tools like Jujutsu or git-branchless). Patrick Steinhardt proposed an **"oplog"** (operations log) to record high-level Git operations (e.g., "squash commits A..B") alongside their ref updates, enabling commands like `git undo`.

**Key points:**
- **Short-term**: Improved documentation or a `--dry-run --verbose` mode may mitigate the issue.
- **Long-term**: The oplog idea could address recoverability for many Git commands, not just `squash`.
- **Template debate resolved**: The `--reedit-message` template now uses a **minimalist design** (numbered subject list + editable body, `squash!` bodies retained, `fixup!`/`amend!` messages omitted) to reduce clutter.

**Next steps:** Junio’s final review; oplog discussion may inspire future work.

---

#### **3. Memory leaks in hash implementations (9-patch series)**
**Headline:** *Leaks in OpenSSL/libgcrypt backends plugged; series ready to merge*
**Author:** Jeff King (Peff)
**Status:** **v1 complete**, with Junio’s approval for patch 1/9.

Peff’s series addresses **memory leaks** in Git’s hash implementations when using non-default backends (e.g., OpenSSL SHA-256, libgcrypt). The leaks were discovered via `SANITIZE=leak` and affect low-level plumbing like `csum-file`, `patch-id`, and HTTP object requests. The fix introduces `git_hash_discard()` and systematically applies it across subsystems, with platform-specific optimizations in the final patch.

**Key changes:**
- **Patch 2/9** adds `git_hash_discard()` (a safe cleanup primitive).
- **Patches 3–8/9** apply the fix to leaky subsystems (e.g., `csum-file`, `http.c`).
- **Patch 9/9** replaces the "hacky" `git_hash_discard()` with platform-specific discard functions.

**Impact:** Leaks only affect non-default hash backends; no user-visible behavior changes.

---

#### **4. CI/test infrastructure for `GIT_TEST_LONG` (9-patch series)**
**Headline:** *Broken tests fixed; GitLab CI now runs expensive tests*
**Authors:** Toon Claes, Patrick Steinhardt
**Status:** **v1 complete**, with **substantive review** from SZEDER Gábor and Jeff King.

The series makes `GIT_TEST_LONG` tests **reliable and efficient** for CI by:
1. **Fixing broken tests** (e.g., `t0021-conversion.sh` on 64-bit systems).
2. **Improving efficiency** (e.g., replacing `dd` with `genzeros` in `t4141-apply-too-large.sh`).
3. **Enabling `GIT_TEST_LONG` in GitLab CI** (matching GitHub Actions).

**Key feedback:**
- **SZEDER Gábor** noted that patch 3/9 (removing `EXPENSIVE` from `t4141`) doesn’t address high memory usage (1 GiB RSS), suggesting the prerequisite should be retained.
- **Jeff King** questioned the portability of `test_copy_bytes` (patch 3/9) and the determinism of patch 5/9’s `SIZE_T_IS_64BIT` prerequisite.

**Next steps:** Authors to address feedback; series otherwise ready.

---

#### **5. Reftable security hardening (v2)**
**Headline:** *Fuzzing infrastructure added; all fixes now present*
**Author:** Patrick Steinhardt
**Status:** **v2 complete**, with Christian Couder’s test-helper feedback addressed.

The series hardens Git’s reftable backend against **corrupted files** (e.g., out-of-bounds reads, NULL dereferences) using **libFuzzer** and **Meson build support**. All 12 patches are now present, including a new test helper (`cl_reftable_write_block`) to reduce boilerplate in unit tests.

**Key changes:**
- **Patches 1–2/12**: Fuzzing infrastructure (Meson + libFuzzer).
- **Patch 5/12**: Test helper for block corruption scenarios.
- **Patches 6–12/12**: Individual security fixes with unit tests.

**Next steps:** Junio’s review pending; no open questions.

---

### **In brief**
- **`git rev-list --exclude-first-parent-only` bug**: Michael Hore reported unexpected behavior when additional commits are specified; likely a logic flaw in `revision.c`.
- **`git apply` memory leak**: Zephyr Yao fixed a leak in `find_header()` where abandoned Git-style diff headers corrupted state.
- **`git rm -n *.json` recursion**: Евгений Плискин reported unintended recursion on Windows; may be a core Git issue.
- **Meson build race fix**: D. Ben Knoble’s patch to fix `hook-list.h` generation was approved by Adrian Ratiu and Patrick Steinhardt.
- **ODB refactoring (v2)**: Patrick Steinhardt’s series to replace `whence` with `source_infop` is ready, with naming feedback addressed.
- **Git v2.55.0**: Weijie Yuan humorously noted "Claude Sonnet 4.6" in the contributor list.
- **Git for Windows 2.55.0(2)**: Johannes Schindelin released a hotfix re-enabling NTLM authentication (temporarily).
- **Git Rev News #136**: Christian Couder announced the latest edition.

---

### **On the radar**
- **Oplog for recoverability**: Patrick Steinhardt’s proposal to add an operations log for atomic undo (long-term).
- **`test_copy_bytes` modernization**: Jeff King’s curiosity about portability assumptions (follow-up to patch 3/9 in the `GIT_TEST_LONG` series).
- **Reftable performance**: Kristofer Karlsson’s merged fix for quadratic-time behavior, but redundant `stat()` calls remain unaddressed.