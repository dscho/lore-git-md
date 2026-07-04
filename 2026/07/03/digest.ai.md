Here’s the digest for **2026/07/03**, covering the Git mailing list traffic:

---

### **The day in brief**
A busy but focused day on the Git mailing list, with **106 emails across 26 threads**. The standout themes:
1. **Security hardening** (reftable backend fixes, memory leaks in hash implementations).
2. **Performance and correctness** (worktree diff slowdowns, `git replay --linearize` regression fixes).
3. **Test infrastructure** (linting, CI improvements, and `GIT_TEST_LONG` reliability).
4. **CLI modernization** (new `git refs` subcommands, `git history` signing, and a proposal to replace `--cached` with `--index`).

The most consequential thread today was **Patrick Steinhardt’s reftable security-hardening series (v3)**, which systematically fixes vulnerabilities in Git’s reftable backend. Meanwhile, **Jeff King’s memory-leak fixes** for non-default hash implementations and **Toon Claes’s `git replay --linearize` regression fixes** highlight the project’s ongoing focus on robustness. On the user-facing side, **Patrick’s `git refs` subcommands** and **Souma’s `git history` signing patches** advance Git’s usability.

---

### **Notable threads**

#### **Reftable security hardening (v3)**
**Headline**: Patrick Steinhardt’s 12-patch series hardens Git’s reftable backend against maliciously corrupted files, fixing out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. The series includes new fuzzing infrastructure (libFuzzer + Meson) and test helpers to prevent regressions.
**Status**: All feedback addressed; Junio C Hamano has approved several patches as "obviously correct." The series is technically complete and ready for merging.
**Why it matters**: Reftable is a critical backend for large repositories, and this hardening effort ensures it’s resilient against local corruption (e.g., disk errors or malicious actors). The fuzzing infrastructure is a reusable asset for future security work.

#### **Memory leaks in non-default hash implementations**
**Headline**: Jeff King’s 9-patch series plugs memory leaks in Git’s hash context (`git_hash_ctx`) when using OpenSSL or libgcrypt backends. The leaks were invisible with Git’s default hash backends but surfaced under `SANITIZE=leak`.
**Status**: Patch 1/9 (removing a redundant `discard_hashfile()` function) is approved. Reviewers (Patrick Steinhardt, Brian M. Carlson) have raised design questions about making `git_hash_discard()` idempotent and renaming it to `git_hash_release()` for consistency with Git’s resource-cleanup conventions.
**Why it matters**: While the leaks are niche (only affecting non-default backends), they’re a correctness issue that could cause subtle bugs in long-running processes. The discussion also touches on Git’s Rust integration, as idempotent cleanup would simplify Rust bindings.

#### **`git replay --linearize` regression fixes (v6)**
**Headline**: Toon Claes’s 3-patch series fixes critical regressions in `git replay --linearize` (silent commit dropping, merge commit divergence) introduced in v5. The series also clarifies the design intent: `--linearize` produces a single linear sequence, even if this duplicates shared history.
**Status**: Junio C Hamano has accepted the design but raised a process question about authorship (whether Toon should be the sole author of the final patch, given Johannes Schindelin’s co-authorship in v5). The series is ready for re-merge once this is resolved.
**Why it matters**: `git replay` is an experimental but powerful tool for history rewriting. These fixes ensure it behaves predictably, especially for users working with complex merge topologies.

#### **`git refs` subcommands (create/delete/update/rename)**
**Headline**: Patrick Steinhardt’s 5-patch series consolidates reference manipulation under `git refs`, adding `create`, `delete`, `update`, and `rename` subcommands. The series modernizes the API and improves discoverability.
**Status**: Fully merged into `next` after a minor typo fix. Toon Claes raised usability questions about `--no-deref` for symref creation and symref renaming, but these are deferred for future work.
**Why it matters**: This is a long-awaited UX improvement, reducing fragmentation in Git’s reference-handling commands (previously scattered across `git-update-ref`, `git-symbolic-ref`, etc.).

#### **`git history drop` subcommand**
**Headline**: Patrick Steinhardt’s 11-patch series adds `git history drop` to remove a commit and replay its descendants. The series also modernizes the reset API and advances `the_repository` removal.
**Status**: Fully merged after Junio C Hamano fixed a whitespace nit. The series includes 561 lines of test coverage for edge cases (detached HEAD, bare repositories).
**Why it matters**: `git history` is an experimental but promising tool for history editing. This subcommand fills a gap between `git rebase` and `git commit --amend`, offering a more intuitive way to excise commits.

---

### **In brief**

#### **Test infrastructure**
- **`greplint.pl`**: Michael Montalbo’s 6-patch series introduces a linter to convert bare `grep` assertions to `test_grep`, improving debuggability. The series fixes 10+ pre-existing bugs and adds documentation for `test_grep`.
- **`GIT_TEST_LONG` reliability**: Patrick Steinhardt’s 9-patch series makes `GIT_TEST_LONG` tests reliable and efficient for CI, reducing disk usage and runtime. The series enables `GIT_TEST_LONG` in GitLab CI and skips broken tests on unsupported platforms.
- **`t9811` modernization**: Marcelo Machado Lage’s patch replaces legacy shell test constructs with modern helpers (`test_path_is_file`, `test_path_is_missing`). Patrick Steinhardt requested minor improvements to the commit message.

#### **Performance and correctness**
- **Worktree diff slowdowns**: Jeff King and D. Ben Knoble investigated a 223.3ms vs. 3.4ms slowdown in `git diff` for secondary worktrees, tracing it to timestamp precision and racy-git scenarios. Peff proposed a minimal patch to enable nanosecond-aware timestamp comparisons unconditionally.
- **`git rev-list --exclude-first-parent-only`**: Junio C Hamano fixed a bug where the option incorrectly excluded commits when additional commits were explicitly specified. The fix adds an early return in `process_parents()` to avoid marking commits as uninteresting.

#### **CLI modernization**
- **`git history` signing**: Souma’s 3-patch series teaches `git history` to sign rewritten commits (`fixup`, `reword`, `split`) using `commit.gpgsign` and `-S/--gpg-sign`. The series adds test coverage and updates documentation.
- **`git repo info` prefix queries**: A GSoC contributor’s patch adds category-based prefix querying to `git repo info` (e.g., `git repo info layout` returns all `layout.*` keys). Junio C Hamano suggested supporting glob patterns (e.g., `layout.*`) for greater expressiveness.
- **Replace `--cached` with `--index`**: A patch proposed deprecating `--cached` in `git diff` in favor of `--index`. The change aims to improve clarity but has not yet gained traction.

#### **Platform compatibility**
- **macOS UTF-8 filenames**: Ihar Hrachyshka’s patch fixes a crash in `git status` on macOS when reading UTF-8 filenames longer than 255 bytes. The fix converts `d_name` to a flexible array member and adjusts allocation logic. Junio C Hamano requested the test be rewritten to avoid Perl.

---

### **On the radar**
- **`USE_NSEC` debate**: D. Ben Knoble and Jeff King’s discussion about whether to flip the default of `USE_NSEC` (nanosecond timestamp support) to `true` remains unresolved. The debate centers on platform compatibility, interoperability risks (e.g., Git/JGit mixing), and whether the knob should be runtime-configurable.
- **`git replay --linearize` authorship**: Junio C Hamano’s question about whether Toon Claes should be the sole author of the final patch in the `git replay --linearize` series is still pending. This is a procedural matter but could delay merging.
- **`git refs` usability**: Toon Claes’s feedback about `--no-deref` for symref creation and symref renaming in `git refs` has not been addressed. These are usability concerns that may inform future refinements.

---

### **Editorial note**
Today’s traffic reflects Git’s dual focus on **robustness** (security hardening, memory leaks, regression fixes) and **usability** (new subcommands, CLI modernization). The reftable security series is particularly noteworthy for its thoroughness and the introduction of fuzzing infrastructure, which will benefit future hardening efforts. Meanwhile, the `git refs` and `git history` subcommands demonstrate the project’s commitment to reducing fragmentation in Git’s interface. The `USE_NSEC` debate and `git replay --linearize` authorship question are the only unresolved loose ends with potential to spill into future discussions.