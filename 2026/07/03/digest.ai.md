Here’s the digest for **2026/07/03**, covering the day’s most consequential developments in Git development:

---

### The day in brief
**July 3, 2026** was a **heavy-traffic day** (106 emails, 26 threads) with **two major milestones** and a **contentious design debate**. The **`git history drop` series** and **reftable hardening patches** reached completion, while a **long-running discussion about `USE_NSEC`** took a sharp turn toward runtime configuration. A **new feature request for `git repo info`** sparked a design debate about glob patterns versus prefix matching, and a **macOS Unicode crash fix** neared readiness. The day’s texture was **technical and forward-looking**, with a mix of **bugfixes, feature work, and infrastructure improvements**.

---

### Notable threads

#### 1. **`git history drop` series merged** – *Patrick Steinhardt*
**Headline**: The 11-patch series adding `git history drop` (remove a commit and replay descendants) is **fully merged** after Junio C Hamano fixed a whitespace nit locally. The feature modernizes the reset API, advances `the_repository` removal, and includes **561 lines of test coverage** for edge cases like detached HEAD and bare repositories.
**Why it matters**: This is a **major addition to Git’s history-editing toolkit**, alongside `git rebase` and `git commit --amend`. The series also **resolved a critical ref-resolution dispute** in `find_head_tree_change()`, ensuring correct HEAD movement detection in detached-HEAD states.
**Next steps**: The feature will ship in Git 2.56. Follow-up work on root/merge commit support and ORIG_HEAD/HEAD edge cases remains non-blocking.

---

#### 2. **Reftable hardening series finalized** – *Patrick Steinhardt*
**Headline**: The 12-patch series hardening Git’s reftable backend against corrupted files is **technically complete** at v3. All prior feedback has been addressed, including fuzzing infrastructure (libFuzzer + Meson support) and a test helper to reduce boilerplate. Junio C Hamano has approved several patches as "obviously correct."
**Why it matters**: Reftable is a **critical backend** for large repositories, and this series **systematically eliminates crashes, infinite loops, and memory corruption** triggered by maliciously crafted files. The fuzzing infrastructure is a **first for Git**, enabling future hardening work.
**Next steps**: Junio is expected to queue the series in `seen` shortly. The fuzzer will run in CI to catch regressions.

---

#### 3. **`USE_NSEC` debate pivots to runtime configuration** – *D. Ben Knoble, Jeff King, Patrick Steinhardt*
**Headline**: The discussion about `USE_NSEC` (sub-second timestamp tracking) has **shifted from build-time parity to runtime configuration**. Patrick Steinhardt proposes **always compiling nanosecond support into Git** (when the platform allows it) and exposing it via a runtime config knob (e.g., `core.useNsec`). This avoids auto-detection challenges while making the feature more accessible.
**Why it matters**: `USE_NSEC` eliminates "racy Git" issues on modern filesystems (ext4, XFS, etc.), but its build-time knob has been **historically underused**. Runtime configuration would **simplify adoption** while retaining a fallback for edge cases.
**Open questions**:
- What should the default for `core.useNsec` be? Peff suggests `false` to avoid breaking interoperability (e.g., Git/JGit mixing).
- Should the build-time knob be deprecated or retained for power users?
**Next steps**: A follow-up patch series is likely, but no timeline has been set.

---

#### 4. **`git repo info` prefix querying sparks design debate** – *GSoC contributor, Junio C Hamano*
**Headline**: A GSoC patch adding **category-based prefix querying** to `git repo info` (e.g., `git repo info layout` returns `layout.bare` and `layout.shallow`) has **drawn a design critique** from Junio C Hamano. He argues for **glob patterns** (e.g., `layout.*`) instead of simple prefix matching, calling globs more expressive and future-proof.
**Why it matters**: The feature addresses **usability friction** as Git’s metadata keys grow, but Junio’s feedback highlights a **tension between simplicity and flexibility** in CLI design. The debate may influence future plumbing command enhancements.
**Next steps**: The author is expected to revise the patch to support globs, possibly using a hybrid exact-match/glob-matcher approach.

---

#### 5. **MacOS Unicode crash fix nears readiness** – *Ihar Hrachyshka*
**Headline**: A patch fixing a crash in `git status` on macOS when reading UTF-8 filenames longer than 255 bytes is **ready for merging** after addressing all feedback. The fix converts `d_name` to a flexible array member in Git’s `precompose_utf8` compatibility layer, eliminating fortified libc overflow checks.
**Why it matters**: This is a **real-world crash** affecting macOS users with long Unicode filenames. The fix is **well-scoped** and includes a regression test.
**Next steps**: Junio is expected to merge the patch after the author rewrites the test to avoid Perl (using `printf` and `tr` instead).

---

### In brief
- **`git replay --linearize`**: Junio C Hamano raised a **procedural nit** about authorship attribution for the v6 series, but the feature remains **technically complete** and under review.
- **`git refs` subcommands**: The series adding `create`, `delete`, `update`, and `rename` to `git refs` is **ready for `next`**, with no further changes needed.
- **`GIT_TEST_LONG` tests**: Patrick Steinhardt’s 9-patch series making expensive tests reliable and CI-friendly is **ready for merging**, with all feedback addressed.
- **Memory leaks in hash implementations**: Jeff King’s 9-patch series plugging leaks in non-default hash backends (OpenSSL, libgcrypt) is **under review**, with a debate about making `git_hash_discard()` idempotent.
- **`git rm` pathspec behavior**: A thread clarifying why `git rm -n *.json` recurses into subdirectories **shifted to documentation**. The behavior is correct but **under-documented**; no code changes are planned.
- **Programmatic commit editing**: A feature request for `git rebase -i` was **resolved by demonstrating `GIT_SEQUENCE_EDITOR`**, which already provides the requested functionality.

---

### On the radar
- **`includeIf.worktree` symlink inconsistency**: Patrick Steinhardt identified a **behavioral discrepancy** between `worktree` and `gitdir` conditions (symlinks are resolved for `worktree` but not for `gitdir`). This must be resolved before the series graduates to `next`.
- **`git history` signing**: Souma’s 3-patch series teaching `git history` to sign rewritten commits is **under review**, with no feedback yet.
- **`--index` vs. `--cached` in `git diff`**: A patch proposing to deprecate `--cached` in favor of `--index` has **drawn surface-level support** but no substantive review. The change may face **backward-compatibility concerns**.

---

### Editorial note
The day’s traffic was **heavily skewed toward infrastructure and hardening**, with **two major series reaching completion** (`git history drop`, reftable hardening) and **one long-running debate taking a decisive turn** (`USE_NSEC`). The **GSoC feature request for `git repo info`** introduced a **design tension** that may recur in future plumbing command enhancements, while the **macOS Unicode crash fix** underscores the importance of platform-specific edge cases. The **memory-leak series** and **`GIT_TEST_LONG` improvements** reflect ongoing efforts to **improve Git’s robustness and CI coverage**. Overall, the day’s work **advances Git’s technical foundation** while surfacing **user-facing design questions** that will shape future releases.