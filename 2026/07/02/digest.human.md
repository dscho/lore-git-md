Here’s the digest for **July 2, 2026**, covering the Git mailing list’s daily traffic:

---

### The day in brief
A **busy but focused** day on the Git mailing list, with **68 emails across 21 threads**. The standout themes:
1. **Post-merge bugfixes** dominated, with Jeff King’s 9-patch series plugging memory leaks in non-default hash backends (OpenSSL/libgcrypt) and a follow-up to brian m. carlson’s merged `rev-parse --parseopt` exit-code fix.
2. **CI and test infrastructure** saw active discussion, including Patrick Steinhardt’s 9-patch series making `GIT_TEST_LONG` tests reliable for GitLab CI and a debate about the `EXPENSIVE` prerequisite’s definition.
3. **Feature finalization**: Toon Claes’s `git replay --linearize` (v6) and Harald Nordgren’s `git history squash` (v7) are now technically complete, with only minor documentation tweaks remaining.
4. **Release notes**: Git 2.55.0’s announcement drew lighthearted attention for its inclusion of "Claude Sonnet 4.6" in the contributor list.

---

### Notable threads

#### **Memory leaks in non-default hash backends (9 patches)**
**What’s happening**: Jeff King (Peff) submitted a **9-patch series** fixing memory leaks in Git’s hash implementations when using OpenSSL or libgcrypt backends. The leaks were discovered via `SANITIZE=leak` and affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. Patch 1/9 (removing a redundant `discard_hashfile()` function) received **maintainer approval** from Junio, while later patches introduce a new `git_hash_discard()` primitive and apply it systematically.
**Why it matters**: These leaks are invisible with Git’s default hash backends but could cause resource exhaustion in long-running processes (e.g., servers) using non-default backends. The series is **ready for merging**, with only minor naming discussions (e.g., `git_hash_discard()` vs. `git_hash_release()`) remaining.
**Key detail**: Patch 9 replaces a "hacky" discard implementation with platform-specific callbacks, eliminating inefficiency.

---

#### **`git replay --linearize` (v6)**
**What’s happening**: Toon Claes posted **v6** of the `--linearize` option for `git replay`, which flattens merge commits into a linear history. The series is now **technically complete**, addressing all prior feedback, including a regression fix for single-branch replay with merges. The interface design debate (standalone `--linearize` vs. `git rebase`-style `--rebase-merges=<mode>`) is unresolved but no longer blocking.
**Why it matters**: This feature provides a simpler alternative to Johannes Schindelin’s earlier merge-replay implementation, with predictable all-or-nothing flattening. The series is **ready for review**, with no open technical objections.
**Key detail**: The behavioral difference from `git rebase --no-rebase-merges` is now documented and tested.

---

#### **`git history squash` (v7)**
**What’s happening**: Harald Nordgren’s **v7** of `git history squash` (collapsing a commit range into its oldest commit) is now **complete**, with all prior feedback addressed. The series includes:
- Stricter input validation (rejecting ranges whose oldest commit is a `fixup!`/`squash!`).
- Adoption of `git rebase -i`’s squash-message template for `--reedit-message`.
- Expanded documentation and test coverage.
**Why it matters**: This feature avoids the repeated conflict stops of a rebase-based approach, making it ideal for large-scale history rewrites. The series is **ready for Junio’s final review**, with only minor usability questions (e.g., whether `--reedit-message` should be the default) remaining.
**Key detail**: A **recoverability discussion** emerged about Git’s reflog limitations for multi-ref operations, with Patrick Steinhardt proposing an "oplog" (operations log) as a long-term solution.

---

#### **`GIT_TEST_LONG` tests for GitLab CI (9 patches)**
**What’s happening**: Patrick Steinhardt’s **9-patch series** makes `GIT_TEST_LONG` tests reliable and efficient enough to run in GitLab CI. The series:
- Fixes broken tests (e.g., `t0021-conversion.sh` on 64-bit systems).
- Improves efficiency (e.g., replacing a 6-minute `dd` loop in `t4141-apply-too-large.sh` with a 1-second `genzeros` call).
- Enables `GIT_TEST_LONG` for GitLab CI pushes to integration branches.
**Why it matters**: GitLab CI previously skipped these tests, leaving long-running test coverage invisible. The series is **ready for merging**, though a debate about the `EXPENSIVE` prerequisite’s definition (runtime vs. resource usage) remains unresolved.
**Key detail**: Patch 3’s removal of the `EXPENSIVE` prerequisite drew **substantive feedback** from SZEDER Gábor, who argued that high memory usage (>1 GiB) still justifies the label.

---

#### **Post-merge follow-ups to `rev-parse --parseopt`**
**What’s happening**: Jeff King (Peff) and brian m. carlson addressed **post-merge issues** in the recently merged `rev-parse --parseopt` exit-code fix. Peff identified:
1. A **shell-quoting bug** in the `SVN` test prerequisite (Perl treats dotted version strings as ASCII, not numeric).
2. A **refactoring gap** in `usage_with_options_internal()` that could cause inconsistent help output.
3. **Behavioral trade-offs** for scripts wrapping Git commands (e.g., `git log --my-options "$@"` now exits 0 on `--help` instead of 129).
**Why it matters**: These follow-ups ensure the fix’s robustness and clarify edge cases. The `SVN` prerequisite issue will be resolved by removing the version check entirely, while the refactoring gap may require a one-line fix.
**Key detail**: The trade-offs are **documented and accepted**, with no further action planned.

---

### In brief
- **Git 2.55.0 released**: Junio announced the final release, drawing attention for including "Claude Sonnet 4.6" in the contributor list.
- **Git for Windows 2.55.0(2)**: Johannes Schindelin released a hotfix re-enabling NTLM authentication as an opt-in deprecated feature.
- **`git rm -n *.json` recursion**: A discussion clarified that the behavior is expected (Git’s pathspecs match `/` by default) and shifted to improving `git-rm(1)`’s documentation.
- **`git rev-list --exclude-first-parent-only` bug**: Junio proposed a fix for a logic flaw when explicit commits are specified.
- **`git apply` memory leak**: A student contribution fixed a leak and state corruption in `find_header()`.
- **Test modernization**: Marcelo Machado Lage updated `t9811-git-p4-label-import.sh` to use modern test helpers (`test_path_is_file`/`test_path_is_missing`).
- **Reftable hardening**: Christian Couder approved Patrick Steinhardt’s v3 of the 12-patch security-hardening series for Git’s reftable backend.
- **ODB refactoring**: Patrick Steinhardt’s `struct object_info` refactoring (6 patches) is **ready for `next`** after a minor reroll.

---

### On the radar
- **Oplog proposal**: Patrick Steinhardt’s vision for an "operations log" to enable atomic undo of multi-ref operations (e.g., `git history squash --update-refs`) remains a long-term architectural discussion.
- **`EXPENSIVE` prerequisite**: The debate about whether it should account for memory usage (not just runtime) may resurface in future test suite discussions.