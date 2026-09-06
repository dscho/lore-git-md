# Git mailing list daily digest for 2026/09/05

## The day in brief

The Git mailing list saw key developments in the `git repo info` path-related keys series, where the author acknowledged and fixed a correctness bug in `path.superproject-root`. CI modernization efforts continued with Jeff King updating workflows to use Debian 12, while a test modernization patch was rejected for misapplying assertion helpers in control-flow contexts. The `--force-if-includes` safety mechanism received a critical bugfix, and the `git cherry-pick --no-commit` documentation series advanced with revived test coverage.

## Notable threads

### `git repo info` path-related keys: correctness bug fix confirmed
**[2026/07/16/01-21-31]**

K Jayatheerth confirmed the fix for a correctness bug in the `path.superproject-root` implementation. The underlying `get_superproject_working_tree()` function was ignoring its `repo` parameter and using `xgetcwd()` instead, which broke expected behavior when the command was invoked from a submodule’s `.git` directory. The fix—updating the function to respect the `repo` parameter and use the repository’s working directory—will be included in v6 of the series, along with additional test coverage for the `--git-dir` case. This resolves the primary blocker for the series, which remains under active review.

The series adds seven new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`) to `git repo info`, exposing filesystem locations of repository components in a user-facing, scriptable format. The architectural concern about shared logic between `git repo info` and `git rev-parse` remains unresolved, but the correctness bug fix takes precedence.

### CI modernization: Debian 12 update and Coccinelle role clarification
**[2026/07/26/08-32-54] [2026/09/05/13-58-22]**

Jeff King updated Git’s CI workflows to use Debian 12 instead of Debian 11, citing Debian 11’s end-of-LTS status. The patch updates `.github/workflows/main.yml` and `.gitlab-ci.yml` to point to `debian:12` and adjusts the support window comment to reflect the new LTS end date (2028-06-30). Junio C Hamano confirmed noticing the same Debian 11 job failures and will queue the patch for integration. The change is mechanical and aligns with the project’s practice of tracking supported LTS distributions.

In a related discussion, Peff clarified Git’s stance on Coccinelle’s role: temporary API-migration rules are not worth the effort, while permanent antipattern-enforcement rules (e.g., style checks, preferred function usage) remain valuable. He also acknowledged the 4.5× performance regression in Coccinelle 1.3.1 as a known trade-off, citing long-term maintainability as the tie-breaker. The discussion remains under review, with no immediate changes proposed to the CI image choice or Coccinelle rule scope.

### `--force-if-includes` bugfix: reflog check corrected
**[2026/09/04/21-01-20]**

Tyler Cipriani posted a two-patch series fixing a critical bug in the `--force-if-includes` safety mechanism. The mechanism was incorrectly checking the reflog of the *local* branch whose name matched the *remote* destination branch (e.g., `main` when pushing to `origin/main`), rather than the branch actually being pushed (e.g., `src` when pushing `src:main`). This flaw could cause false rejections or unintended data loss. The fix updates `remote.c` to consult the reflog of the pushed ref (`ref->peer_ref->name`) and adds special handling for `HEAD`-based pushes and detached HEAD states. D. Ben Knoble provided substantive review, confirming the fix’s behavior and accepting the reflog limitation for detached HEAD as a necessary trade-off.

The series also introduces a new `advice.forceIfIncludesDetachedHead` config knob (documented as `advice.pushRefUnverifiable`) and updates advice messages to be actionable and consistent across transport layers. Eight new test cases in `t/t5533-push-cas.sh` cover mismatched local/remote names, `HEAD`-based pushes, and detached HEAD states.

### `git cherry-pick --no-commit` documentation: test coverage revived
**[2026/09/03/12-55-23]**

Aleksei Sviridkin revived the test patch in the `git cherry-pick --no-commit` documentation series, adding a one-line check (`test_ref_missing CHERRY_PICK_HEAD`) to guard against regressions in the "tricky" logic of `do_pick_commit()`. The test verifies that `CHERRY_PICK_HEAD` is *not* created when a `git cherry-pick --no-commit` operation fails due to conflicts, addressing Phillip Wood’s concern that the existing test suite does not cover this edge case. The documentation patch clarifies this long-standing but undocumented behavior, framing the absence of `CHERRY_PICK_HEAD` as a deliberate design choice.

The test’s inclusion remains contentious. Phillip Wood supports it as a necessary safeguard, while Junio C Hamano and Patrick Steinhardt oppose it on test-suite overhead grounds. Junio also suggested editorial tweaks to the documentation, recommending clearer phrasing about the design intent behind `--no-commit` as a tool for custom work. The series touches only `Documentation/git-cherry-pick.adoc` and `t/t3507-cherry-pick-conflict.sh`, with no code logic or CLI changes.

### Test modernization: `test -f` → `test_path_is_file` rejected
**[2026/09/04/20-35-51]**

Junio C Hamano rejected Mark C. Chu-Carroll’s patch series replacing `test -f` with `test_path_is_file` in 63 test scripts. The mechanical replacement was incorrect in control-flow contexts (e.g., checking for platform-specific files like `/proc/$shell_pid/winpid`), where the helper functions are designed for assertions, not silent conditionals. The patch would turn silent checks into loud assertion failures, altering test semantics. Junio emphasized that each `test -f` must be analyzed for its semantic purpose rather than replaced mechanically. The series is now blocked until the approach is rethought.

## In brief

- **[2026/09/02/08-31-37]** Thomas Bachem will remove the v3 special-case behavior in the `rerere` race-condition fix, ensuring all commands (except `git rerere gc`) wait the full configurable timeout and fail if the lock is still held. Junio insisted on loud failures to avoid silently losing the user’s conflict resolution effort.
- **[2026/09/02/13-04-09]** Junio clarified that `common_prefix_len()` returns an empty string (not a bogus string) when the first pathspec item is excluded, refining the bug’s description without altering the fix’s correctness.
- **[2026/09/03/01-05-47]** Aleksei Sviridkin posted v3 of the `--force-if-includes` bugfix, initializing `timestamp_t date` to `0` in `remote.c` and justifying the fallback value as the correct default for ensuring a full reflog scan when no remote-tracking reflog exists.
- **[2026/08/26/10-55-19]** Christian Couder will submit Git’s Outreachy application imminently, proposing two project ideas: continuing the removal of global state (libifying code) and improving command argument and option parsing.
- **[2026/09/01/08-16-11]** Jeff King and Todd Zullinger added context about Asciidoctor version compatibility, noting that even older enterprise distributions (Debian 11, RHEL 8) ship Asciidoctor 2.x, reinforcing the decision to drop the outdated version pin in CI.
- **[2026/09/04/07-53-44]** Thomas Bachem will store the cached `GIT_CONFIG_PARAMETERS` string as a `const char *` in the sequencer auto-maintenance deferral series, clarifying immutability.
- **[2026/09/04/23-55-48]** Junio asked whether a prior reply about "diff with submodules" signals that a reroll of `dk/use-nsec-runtime` is expected. Kristoffer Haugsbakk requested that `kh/doc-datamodel` not be merged to `next` yet, pending a reroll to address Julia Evans’s feedback.
- **[2026/09/05/16-29-34]** Fabian Kratz reported a typo in the `git-merge(1)` man page, claiming `ORIG_HEAD` is incorrectly documented as pointing to `C` instead of `G`. Kristoffer Haugsbakk confirmed the issue was fixed in Git 2.51.0.
- **[2026/09/05/20-26-04]** Erdhoernchen asked how to integrate a local commit from a development repository’s orphan "release" branch into a GitHub PR branch in a dispatch repository, ensuring GitHub recognizes the PR as resolved without introducing extra merge commits.