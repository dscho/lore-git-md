Here’s the daily digest for **July 2, 2026** (UTC), covering the Git mailing list traffic:

---

### The day in brief
A **moderately busy day** (68 emails, 21 threads) with a mix of **bugfixes, feature refinements, and CI/test infrastructure work**. The standout themes:
1. **Memory leak fixes** (Jeff King’s 9-patch series for hash backends) and **test suite modernization** (Toon Claes/Patrick Steinhardt’s 9-patch CI series) dominated the volume.
2. **`git replay --linearize`** (Toon Claes) and **`git history squash`** (Harald Nordgren) reached technical completion, with interface debates settled.
3. **Git 2.55.0** was released, and Git for Windows shipped a hotfix for NTLM authentication.
4. **Documentation gaps** (e.g., `git rm` pathspec behavior) and **build system fixes** (Meson race condition) rounded out the day.

---

### Notable threads

#### 1. **Memory leaks in non-default hash backends** *(jk/hash-leak-fixes, 9 patches)*
**What’s new**: Jeff King (Peff) posted a **9-patch series** plugging memory leaks in Git’s hash implementations (OpenSSL, libgcrypt) when using non-default backends. The leaks were discovered via `SANITIZE=leak` and affect subsystems like `csum-file`, `patch-id`, and HTTP object requests.
**Key details**:
- Introduces `git_hash_discard()` (later renamed to `git_hash_release()` per Patrick Steinhardt’s feedback) to explicitly free hash contexts.
- Patch 9 replaces a "hacky" dummy-buffer implementation with **platform-specific discard functions**, eliminating inefficiency.
- **Idempotency debate**: Patrick and Peff agreed to enforce strict idempotency (e.g., `BUG()` on misuse) in a follow-up patch.
**Status**: **Ready for `next`**. Junio approved patch 1/9, and the rest are uncontroversial. The series is a **critical correctness fix** for non-default hash users.

---

#### 2. **`GIT_TEST_LONG` tests: Reliability and CI coverage** *(tc/ci-git-test-long, 9 patches)*
**What’s new**: Patrick Steinhardt posted **v3** of a 9-patch series making `GIT_TEST_LONG` tests reliable and efficient enough for CI. The series:
- Fixes **broken tests** (e.g., `t0021-conversion.sh` on 64-bit systems, `t7508-status.sh` on 32-bit).
- **Reduces resource usage** (e.g., `t4141-apply-too-large.sh` now runs in <1s vs. 6 minutes).
- **Enables `GIT_TEST_LONG` in GitLab CI** (matching GitHub Actions) and disables it on Windows runners (RAM limitations).
**Debate**: SZEDER Gábor objected to removing the `EXPENSIVE` prerequisite from `t4141` (memory usage >1 GiB), arguing it’s still "expensive" despite the runtime fix. Peff endorsed the patch but questioned the `test_copy_bytes` helper’s design.
**Status**: **Ready for `next`**. All feedback addressed; the `EXPENSIVE` label was restored in `t4141`.

---

#### 3. **`git replay --linearize`** *(tc/replay-linearize, v6)*
**What’s new**: Toon Claes posted **v6** of the `--linearize` option for `git replay`, which flattens merge commits into a linear history. The series is now **technically complete**:
- **Regression fix**: Restored the `replayed_base` parameter to ensure correct topology when replaying a single branch with merges.
- **Interface debate settled**: Uses `--linearize` (not `git rebase`’s `--rebase-merges=<mode>`), with explicit justification in the commit message.
- **Test coverage**: Added scenarios for replaying multiple branches and verifying the behavioral difference from `git rebase --no-rebase-merges`.
**Status**: **Under review**. No unresolved technical objections; Junio’s procedural nit about authorship attribution remains.

---

#### 4. **`git history squash`** *(hn/history-squash, v7)*
**What’s new**: Harald Nordgren’s **v7** of `git history squash` (folding a commit range into its oldest commit) is **complete and ready for Junio’s final review**:
- **Template alignment**: Adopted `git rebase -i`’s squash-message template (resolving Phillip Wood’s usability critique).
- **Merge handling**: Rejects merges with external parents but allows fully contained merges (Patrick Steinhardt’s design).
- **Recoverability debate**: Phillip Wood and Patrick Steinhardt discussed Git’s **lack of atomic undo** for multi-ref operations (e.g., `--update-refs`), proposing an **oplog** as a long-term solution.
**Status**: **Ready for `next`**. All prior feedback addressed; the recoverability gap is acknowledged but not blocking.

---

#### 5. **Git 2.55.0 released**
**What’s new**: Junio announced **Git 2.55.0**, integrating 505 commits from 100 contributors. Highlights:
- **New features**: Parallel hooks, `git format-rev`, `git url-parse`, Linux fsmonitor daemon.
- **Performance**: Revision traversal, reachability bitmaps, sparse-index optimizations.
- **Internal refactoring**: ODB abstraction, MIDX compaction, Rust readiness.
- **Fun fact**: The contributor list included "Claude Sonnet 4.6" (an AI model), sparking lighthearted commentary.
**Status**: **Released**. No follow-up discussion.

---

### In brief
- **`git rm` pathspec behavior**: Евгений Плискин reported that `git rm -n *.json` recursively matches subdirectories, contrary to expectations. The behavior is **correct** (Git’s pathspecs match `/` by default) but **underdocumented**. Phillip Wood and Mikael Magnusson clarified the `:(glob)` modifier as a workaround.
- **Meson build race**: D. Ben Knoble’s patch to fix a race condition in `hook-list.h` generation was **approved by Adrian Ratiu and Patrick Steinhardt**. Ready for merging.
- **Test modernization**: Marcelo Machado Lage’s patch to modernize `t9811-git-p4-label-import.sh` received **feedback from Patrick Steinhardt** (commit message clarity, readability). Awaiting reroll.
- **Git for Windows 2.55.0(2)**: Johannes Schindelin released a **hotfix** re-enabling NTLM authentication as an opt-in deprecated feature (planned removal in 2026). Also marked the final version to support Windows 8.1.
- **`git rev-list --exclude-first-parent-only`**: Michael Hore reported a bug where explicit commits on the command line cause incorrect exclusions. Junio proposed a **fix targeting `process_parents()`** in `revision.c`.
- **`git apply` memory leak**: Zephyr Yao’s patch fixes a leak in `find_header()` when Git-style diff headers are rejected. **Under initial review**.

---

### On the radar
- **`git history squash` recoverability**: Patrick Steinhardt’s **oplog proposal** (inspired by Jujutsu) could address Git’s lack of atomic undo for multi-ref operations. No immediate action, but worth tracking.
- **`git replay --linearize` authorship**: Junio’s nit about whether Toon Claes or Johannes Schindelin should be the primary author may need clarification before merging.
- **`git rm` documentation**: The pathspec behavior debate highlights a need to **cross-reference `gitglossary(7)`** or clarify the `:(glob)` modifier in `git-rm(1)`.

---

### Editorial note
Today’s traffic leaned toward **infrastructure and correctness** over flashy features. The **memory leak series** and **`GIT_TEST_LONG` CI work** reflect the project’s ongoing investment in robustness, while **`git replay` and `git history`** show steady progress on user-facing tools. The **Git 2.55.0 release** and **Git for Windows hotfix** underscore the project’s maturity and responsiveness to regressions.