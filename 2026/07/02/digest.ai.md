Here’s the daily digest for **July 2, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A busy day with **68 emails across 21 threads**, dominated by **bugfixes, refactoring, and CI improvements**. Key highlights:
- **Memory leaks plugged** in Git’s hash implementations (OpenSSL/libgcrypt) and `git format-patch`.
- **`git replay --linearize` v6** posted, addressing prior feedback and fixing a regression.
- **`git history squash` v7** finalized, with debate shifting to **recoverability** and **template formatting**.
- **Git v2.55.0 released**, with lighthearted commentary on an AI’s inclusion in the contributor list.
- **CI and test suite improvements**, including fixes for `GIT_TEST_LONG` tests and a Meson build race.

---

### **Notable threads**

#### **1. `git replay --linearize` v6: Flattening merges with precision**
**Headline**: Toon Claes posted v6 of the `--linearize` option for `git replay`, which flattens merge commits into a linear history. The series is now **technically complete**, addressing all prior feedback, including a regression fix for single-branch replay with merges. The interface design debate (standalone `--linearize` vs. `git rebase`-style `--rebase-merges=<mode>`) remains unresolved but is no longer blocking. Junio’s test to lock in the behavioral difference from `git rebase --no-rebase-merges` is now included.
**Status**: Ready for review; no unresolved technical objections.
**Key detail**: The `replayed_base` parameter was restored to ensure all commits are linearized into a single topology.

---

#### **2. `git history squash` v7: Finalized, but recoverability questions linger**
**Headline**: Harald Nordgren’s v7 of `git history squash` is **code-complete**, with all prior feedback addressed, including:
- Adoption of `git rebase -i`’s squash-message template.
- Rejection of ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`.
- Expanded documentation and test coverage.
**Debate**: The thread pivoted to **recoverability**—how to undo multi-ref operations like `--update-refs`. Patrick Steinhardt proposed an **oplog (operations log)** to enable atomic undo, while Junio clarified that the reflog was never designed for this purpose. The discussion is forward-looking and doesn’t block the series.
**Status**: Ready for Junio’s final review; minor usability tweaks (e.g., default behavior) may follow.

---

#### **3. Memory leaks: Hash implementations and `git format-patch`**
**Headline**: Jeff King (Peff) posted a **9-patch series** plugging memory leaks in Git’s hash implementations (OpenSSL/libgcrypt) and a **2-patch series** fixing a leak in `git format-patch --base`. The hash series introduces `git_hash_discard()` and systematically applies it across subsystems (`csum-file`, `patch-id`, HTTP object requests). Junio approved the first patch, and Patrick Steinhardt suggested renaming `git_hash_discard()` to `git_hash_release()` for idiomatic consistency.
**Status**: Ready for merging; naming tweaks may follow.
**Key detail**: Leaks only affect non-default hash backends (OpenSSL ≥ 3.0, libgcrypt), so impact is limited but critical for correctness.

---

#### **4. Git v2.55.0 released: AI humor and a hotfix**
**Headline**: Junio announced **Git v2.55.0**, summarizing 505 commits from 100 contributors. The release includes new features (parallel hooks, Linux fsmonitor daemon), performance optimizations (revision traversal, sparse-index), and over 50 bugfixes. Lighthearted commentary noted the inclusion of "Claude Sonnet 4.6" in the contributor list.
**Follow-up**: Johannes Schindelin released **Git for Windows 2.55.0(2)**, a hotfix re-enabling NTLM authentication (deprecated but opt-in) and marking the final version to support Windows 8.1.
**Status**: Released; no further action.

---

#### **5. CI and test suite: `GIT_TEST_LONG` reliability and Meson fixes**
**Headline**: Patrick Steinhardt posted v3 of a **9-patch series** making `GIT_TEST_LONG` tests reliable and efficient for CI. Key changes:
- **Test correctness**: Skip broken tests on incompatible platforms (e.g., `SIZE_T_IS_64BIT`).
- **Efficiency**: Reduce disk usage and runtime (e.g., `t4141-apply-too-large.sh` now runs in <1s).
- **CI visibility**: Add GitLab CI badge to `README.md` and enable `GIT_TEST_LONG` in GitLab CI.
**Debate**: SZEDER Gábor argued that `t4141`’s high memory usage (>1 GiB) still justifies the `EXPENSIVE` label, even with the runtime fix.
**Status**: Ready for merging; minor policy questions remain.

**Meson build race**: D. Ben Knoble’s patch to fix a race in generating `hook-list.h` was **approved by Adrian Ratiu and Patrick Steinhardt**. The fix ensures the header is generated before `builtin/bugreport.c` is compiled.
**Status**: Ready for merging.

---

### **In brief**
- **`git rev-list --exclude-first-parent-only` bug**: Michael Hore reported a misbehavior with explicit commits; Junio proposed a fix targeting `process_parents()` in `revision.c`.
- **`git rm -n *.json` recursion**: Евгений Плискин’s report revealed that `*` in pathspecs matches `/` by default, leading to unexpected recursion. The thread shifted to **documentation improvements** for `git-rm(1)`.
- **`git apply` memory leak**: Zephyr Yao fixed a leak in `find_header()` where abandoned Git-style diff headers corrupted patch state.
- **Reftable hardening**: Christian Couder approved Patrick Steinhardt’s v3 of the **12-patch security-hardening series** for Git’s reftable backend.
- **ODB refactoring**: Patrick Steinhardt’s **6-patch series** refactoring `struct object_info` to use `source_infop` is **ready for `next`** after a minor reroll.
- **Test modernization**: Marcelo Machado Lage modernized `t9811-git-p4-label-import.sh` to use `test_path_is_file`/`test_path_is_missing`; Patrick Steinhardt requested minor improvements.

---

### **On the radar**
- **Recoverability**: Patrick Steinhardt’s **oplog proposal** for atomic undo of multi-ref operations (e.g., `git history squash --update-refs`).
- **`git replay --linearize` interface**: The debate over `--linearize` vs. `--rebase-merges=<mode>` may resurface during review.
- **`GIT_TEST_LONG` policy**: Whether "expensive" should account for memory usage (not just runtime) remains open.