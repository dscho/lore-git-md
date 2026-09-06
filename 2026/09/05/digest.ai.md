# Git mailing list daily digest for 2026/09/05

## The day in brief
K Jayatheerth confirmed the fix for a correctness bug in the `path.superproject-root` implementation for `git repo info`, clearing the way for v6 of the path-related keys series. Jeff King clarified Git’s stance on Coccinelle’s role, distinguishing between temporary API-migration rules (not worth the effort) and permanent antipattern-enforcement rules (valuable). Aleksei Sviridkin posted v3 of the `--force-if-includes` bugfix, addressing an uninitialized variable that caused premature reflog walk termination. The test-coverage debate for `git cherry-pick --no-commit` intensified, with the author reviving a minimal test patch to guard against regressions in the "tricky" logic of `do_pick_commit()`.

## Notable threads

### [PATCH v5] series adding path-related keys to `git repo info`
**What changed**: K Jayatheerth confirmed the fix for the `path.superproject-root` correctness bug: update `get_superproject_working_tree()` to respect the `repo` parameter and use the repository’s working directory instead of `xgetcwd()` (2026/09/05/10-01-14). The fix will be included in v6, along with additional test coverage for the `--git-dir` case.

**Background**: The series adds seven new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`) to `git repo info`, exposing filesystem locations of repository components in a user-facing, scriptable format. The `path.superproject-root` implementation was blocked by a correctness bug where the function ignored its `repo` parameter and relied on the process’s current working directory, breaking expected behavior when invoked from a submodule’s `.git` directory.

**Why it matters**: The fix resolves the primary blocker for the series, which aims to provide a stable, scriptable interface for accessing repository component paths without requiring direct access to Git’s internals. The new keys complete the relative-path set by adding `path.cdup` (the inverse of `path.git-prefix`), addressing prior design objections.

---

### CI: bump Ubuntu image for static-analysis job
**What changed**: Jeff King clarified Git’s stance on Coccinelle’s role, distinguishing between temporary API-migration rules (not worth the effort) and permanent antipattern-enforcement rules (valuable) (2026/09/05/13-44-24). He also acknowledged the 4.5× Coccinelle performance regression as a known trade-off, citing long-term maintainability as the tie-breaker, and agreed the current CI setup installs unnecessary build dependencies but did not propose a concrete plan to address it (2026/09/05/13-52-59).

**Background**: The thread began with a patch series to bump the Ubuntu image for Git’s static-analysis CI jobs from `ubuntu-22.04` to `ubuntu-latest` (resolving to `ubuntu-24.04`), with the goal of picking up a newer Coccinelle version (1.3.0+). SZEDER Gábor’s benchmarks showed Coccinelle 1.3.1 as 4.5× slower than 1.1.1, contradicting the expected performance improvement. The discussion evolved to clarify Coccinelle’s role in Git’s development process.

**Why it matters**: The project now broadly agrees that Coccinelle should focus on permanent antipattern-enforcement rules (e.g., catching `if (a == NULL)` or enforcing `oidclr()` over `hashclr()`) rather than temporary API migrations. This clarification frames the performance regression as an acceptable trade-off for long-term maintainability. The thread also highlighted inefficiencies in CI job setup (unnecessary build dependencies), though no concrete plan to address them was proposed.

---

### [PATCH] push: fix uninitialized variable in `--force-if-includes` reflog walk
**What changed**: Aleksei Sviridkin posted v3 of the `--force-if-includes` bugfix, initializing `timestamp_t date` to `0` in `remote.c` to ensure the reflog traversal covers the entire local branch history when no remote-tracking reflog exists (2026/09/05/17-13-30). The commit message now justifies the fallback value (`0`) as the correct default for forcing a full reflog scan, addressing Junio’s prior feedback.

**Background**: The `--force-if-includes` safety mechanism in `git push` contains an uninitialized variable bug in the files backend that causes the reflog walk to terminate prematurely when the remote-tracking ref has no reflog (e.g., in fresh clones). This leads to false positives in the "remote ref updated since checkout" check, potentially allowing unsafe force-pushes. The bug was introduced in 2020 (commit 99a1f9ae10).

**Why it matters**: The fix is minimal (one line in `remote.c`) and well-tested, addressing a real edge case in a security-sensitive code path. The reflog backend is unaffected, and the new test in `t/t5533-push-cas.sh` ensures the regression cannot recur. The patch is ready for integration, with no remaining technical objections.

---

### [PATCH v3 0/2] git-cherry-pick.txt: document absence of CHERRY_PICK_HEAD on conflicted --no-commit
**What changed**: Aleksei Sviridkin revived the test patch in v3, adding a one-line check (`test_ref_missing CHERRY_PICK_HEAD`) to guard against regressions in the "tricky" logic of `do_pick_commit()` (2026/09/05/17-13-30-1). The test directly addresses Phillip Wood’s argument that the absence of `CHERRY_PICK_HEAD` in conflicted `--no-commit` cherry-picks is not currently verified by the test suite. Junio C Hamano suggested the documentation should better communicate that the absence of `CHERRY_PICK_HEAD` is a deliberate design choice, framing `--no-commit` as a tool for custom work where users are expected to manually handle the working tree afterward (2026/09/05/16-29-51).

**Background**: The series documents the long-standing but undocumented edge case in `git cherry-pick --no-commit` where `CHERRY_PICK_HEAD` is *not* created when the operation fails due to conflicts. The behavior was introduced in 2011 (commit d7e5c0cbfb) and is confirmed as intentional by Phillip Wood, but it remained undocumented until now. The test-coverage debate centers on whether the minimal test patch justifies its inclusion given the broader policy concern about test-suite overhead.

**Why it matters**: The documentation patch resolves a real (if niche) user confusion point, while the test patch’s fate hinges on whether its value (guarding against regressions in subtle logic) outweighs its cost (runtime and maintenance overhead). The series is narrowly scoped and well-motivated, with no unresolved technical concerns.

---

### [PATCH 0/2] Fix --force-if-includes to check the correct reflog
**What changed**: D. Ben Knoble confirmed the `--force-if-includes` fix’s behavior, conceding that requiring an explicit bypass (via `--no-force-if-includes`) is appropriate for a mechanism intended as a reminder to check, not a hard block (2026/09/05/18-57-03). He also accepted the series’ decision to reject detached HEAD pushes by default, calling it "all we can do for now" given the reflog limitation (2026/09/05/18-59-25).

**Background**: The series fixes a bug where `--force-if-includes` incorrectly checks the reflog of the *local* branch whose name matches the *remote* destination branch (e.g., `main` when pushing to `origin/main`), rather than the branch actually being pushed (e.g., `src` when pushing `src:main`). This flaw can cause false rejections or unintended data loss. The fix consults the reflog of the pushed ref (`ref->peer_ref->name`) and rejects detached HEAD pushes by default (configurable via `advice.forceIfIncludesDetachedHead`).

**Why it matters**: The fix addresses a real-world edge case in a security-sensitive code path, ensuring `--force-if-includes` works as intended for mismatched local/remote branch names and detached HEAD states. The advice message improvements make the failure modes actionable and consistent across transport layers.

---

### Modernize test scripts: replace `test -f` with `test_path_is_file`
**What changed**: Junio C Hamano rejected the test modernization patch as submitted, identifying incorrect conversions of `test -f` in control-flow contexts (e.g., platform-specific file checks) to `test_path_is_file`, which is designed for assertions, not silent conditionals (2026/09/05/02-17-10).

**Background**: The series proposed replacing older shell test idioms (`test -f`, `! test -f`, `test -d`) with more semantic helper functions (`test_path_is_file`, `test_path_is_missing`, `test_path_is_dir`) in 63 test scripts under `t/`. The goal was syntactic modernization, aligning with the project’s test guidelines.

**Why it matters**: The rejection highlights a fundamental misunderstanding of the helper functions’ purpose. The patch contained more incorrect conversions than correct ones, particularly in control-flow logic where failure is *not* an error (e.g., checking for `/proc/$shell_pid/winpid` on MINGW). The author must rethink the approach, distinguishing between `test -f` used for assertions (where `test_path_is_file` is appropriate) and `test -f` used for control flow (where it is not).

---

## In brief
- **Outreachy December 2026 cohort**: Christian Couder will submit Git’s application imminently, proposing two project ideas: (1) continuing removal of global state (libifying code), and (2) improving command argument and option parsing by porting ad-hoc code to established APIs (2026/09/05/08-26-34).
- **CI: Asciidoctor version pinning**: Jeff King added that even older enterprise distributions (Debian 11, RHEL 8) ship Asciidoctor 2.x, reinforcing the decision to drop the outdated version pin in CI (2026/09/05/14-02-17). Todd Zullinger noted Asciidoctor in RHEL and clones is packaged in EPEL (not the base OS) and currently unmaintained (2026/09/05/15-23-00).
- **rerere: fix race condition between rebase and background maintenance**: Thomas Bachem will remove the v3 special-case behavior that let foreground commands proceed without `rerere` after a timeout; all commands (except `git rerere gc`) will now wait the full configurable timeout and fail if the lock is still held (2026/09/05/05-41-38). Junio C Hamano insisted that after waiting the full timeout, commands should fail loudly rather than proceed, to avoid silently losing the user’s conflict resolution effort (2026/09/05/16-10-33).
- **dir: fix common prefix calculation with leading exclude pathspec**: Junio C Hamano clarified that `common_prefix_len()` returns an empty string (not a bogus string) when the first pathspec item is excluded (2026/09/05/16-14-18).
- **sequencer: defer auto maintenance until rebase completion**: Thomas Bachem confirmed he will store the cached `GIT_CONFIG_PARAMETERS` string as a `const char *` instead of a `struct strbuf` for clarity and immutability (2026/09/05/05-42-02).
- **Update CI to use Debian 12 instead of Debian 11**: Jeff King updated CI to use Debian 12 instead of Debian 11, citing Debian 11’s end-of-LTS status (2026/09/05/13-58-22). Junio C Hamano will queue the patch (2026/09/05/16-32-45).
- **typo in git-merge(1) man page example**: Fabian Kratz reported a typo in the `git-merge(1)` man page, claiming `ORIG_HEAD` is incorrectly documented as pointing to `C` (tip of current branch) instead of `G` (tip of branch being merged) (2026/09/05/16-29-34). Kristoffer Haugsbakk confirmed the issue was fixed in Git 2.51.0 (commit 953049ee) (2026/09/05/17-00-47).
- **"What's cooking in git.git" report for September 2026 (#02)**: Junio C Hamano asked whether a prior reply about "diff with submodules" signals that a reroll of `dk/use-nsec-runtime` is expected (2026/09/05/23-57-01). Kristoffer Haugsbakk requested to delay merging `kh/doc-datamodel` to `next` pending a reroll to address Julia Evans’s feedback (2026/09/05/06-29-48), and signaled intent to revise the documentation for the retracted `kh/format-patch-range-diff-notes` series (2026/09/05/06-31-47).
- **Workflow question about integrating GitHub pull requests**: Erdhoernchen asked how to integrate a local commit from a development repository’s orphan "release" branch into a GitHub PR branch in a dispatch repository, ensuring GitHub recognizes the PR as resolved without introducing extra merge commits (2026/09/05/20-26-04).