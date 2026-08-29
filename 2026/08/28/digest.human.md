# Git mailing list daily digest for 2026/08/28

## The day in brief
The Git mailing list saw significant progress on several fronts today. Junio C Hamano marked Yoichi NAKAYAMA’s worktree repair patch for `next`, resolving a long-standing asymmetry in `.git` file path handling. Elijah Newren’s five-patch series extending protection against unsafe commit operations during conflict resolution reached v3, addressing all substantive feedback. Meanwhile, Junio proposed a sweeping architectural change to built-in command interfaces, replacing `struct repository *` parameters with a `bool has_repo` flag to improve safety and clarify design intent.

## Notable threads

### Worktree repair bugfix graduates to `next`
**What changed:** Junio C Hamano marked Yoichi NAKAYAMA’s v4 patch for the `next` branch, fixing a bug in `git worktree repair` where relative paths in `.git` files were not correctly detected when `worktree.useRelativePaths` was toggled.

**Problem:** The existing code failed to detect mismatches between the configured path style and the `.git` file contents because `read_gitfile_gently()` always returned an absolute path, obscuring the original path style. This asymmetry prevented `git worktree repair` from symmetrically converting cross-references when the configuration changed.

**Impact:** The patch introduces `read_gitfile_raw()`, a new function that reads the path from a `.git` file *without* resolving it to an absolute path or validating the repository. This allows the worktree repair logic to detect whether the stored path was originally relative or absolute, enabling symmetric conversion when the configuration changes. The fix touches `setup.c`, `setup.h`, `worktree.c`, and test script `t/t2406-worktree-repair.sh`.

**Today’s development:** Junio’s merge signal confirms the patch is ready for integration testing. The expanded commit message in v4 now clearly justifies why `read_gitfile_raw()` omits repository validation, addressing Junio’s earlier feedback that the v3 message was "too sketchy." The patch is technically complete and uncontroversial, with no known loose ends.

---

### Unsafe commit operations during conflict resolution: v3 posted
**What changed:** Elijah Newren posted v3 of a five-patch series that extends Git’s protection against unsafe commit operations during conflict resolution to cover both `git commit --amend` and `git commit <paths>` for `git am`, `git revert`, and all forms of `git rebase`.

**Problem:** The existing protection only covered merges and cherry-picks, leaving users vulnerable to accidentally amending the wrong commit or updating individual paths while resolving conflicts in other commands. The series also addresses a usability issue where error messages were misleading or overly generic.

**Impact:** The series blocks unsafe operations during conflict resolution while preserving legitimate workflows (e.g., interactive rebase directives like `squash` or `edit`). It introduces a new `sequencer_ongoing_operation()` helper to consolidate logic for detecting active operations and distinguishing between conflict stops and clean rebase stops. The patches touch `builtin/commit.c`, `sequencer.c`, `sequencer.h`, and test coverage in `t3404-rebase-interactive.sh`, `t3507-cherry-pick-conflict.sh`, and `t4151-am-abort.sh`.

**Today’s development:** The v3 series addresses all substantive feedback from earlier iterations:
- Patch 1/5 renames `FROM_REBASE_PICK` to `FROM_REBASE_EMPTY` and `is_from_rebase()` to `is_from_rebase_empty()` to clarify intent.
- Patch 2/5 removes the restriction on `git commit <paths>` when a rebase pick becomes empty, addressing Junio’s concern that the result would still be an empty commit and thus harmless.
- Patches 3-5/5 extend the protection to additional commands and improve error messages. Junio explicitly approved the reworded error message in patch 3/5 ("Much less awkward than the previous round") and the thoroughness of patch 5/5 ("Deliberate omission of 'default:' is a plus").

The series is now technically sound and ready for final review, with no remaining objections or open questions.

---

### Architectural redesign of built-in command interfaces
**What changed:** Junio C Hamano proposed a sweeping architectural change to built-in command interfaces, replacing the `struct repository *` parameter in all built-in commands (`cmd_foo()`) with a `bool has_repo` flag. The proposal also includes an 8-patch series refactoring `git checkout`, `git switch`, and `git restore` to eliminate the monolithic `checkout_main()` helper and extract reusable logic into standalone helpers.

**Problem:** The current design creates a false promise of libification for built-in commands, risks segfaults (e.g., `cd / && git foo -h`), and obscures the architectural boundary between built-ins (which are one-time orchestrators like `main()`) and reusable utility functions. Junio’s proposal redirects the `the_repository` removal effort toward utility code outside `builtin/`, where libification is both feasible and meaningful.

**Impact:** The change affects 135 files and eliminates the inconsistent state where `repo` and `the_repository` were used interchangeably. It also provides a concrete path forward for libification: reusable logic should be factored out of built-in commands into standalone helpers that accept a `struct repository *` parameter, placing all callers (built-ins and external programs) on equal footing. The 8-patch series demonstrates this approach by refactoring the checkout/switch/restore codebase, moving reusable logic (e.g., `post_checkout_hook()`) to top-level files like `checkout.c` and `checkout.h`.

**Today’s development:** Junio clarified the proposal’s motivations and goals in two follow-up emails:
1. The `bool has_repo` design is motivated by **safety and clarity**, not efficiency, and is intended as a **final state** for built-in command interfaces.
2. The libification goal is to extract reusable logic into standalone helpers, not to make built-in commands directly callable from other commands. This resolves the tension between libification and the one-time-initialization constraints of built-ins.

The 8-patch series is explicitly marked as an early draft for discussion, not for merging to `seen`. Junio’s self-review of the final patch (8/8) identified a shortcoming: `post_checkout_hook()` still hardcodes `the_repository` and `the_hash_algo`, which undermines its reusability. The fix (adding a `struct repository *` parameter) is straightforward but raises the bar for correctness.

The proposal has not yet attracted objections from other contributors, but its scale and implications suggest it will require careful review and coordination with the broader `the_repository` removal effort.

---

### Negative pathspec handling in `git ls-files` and `git add`
**What changed:** Diogo Castro posted a bugfix patch addressing a long-standing issue where negative pathspecs (exclusions) were incorrectly affected by the common prefix of positive pathspecs. This could cause negative pathspecs to degenerate into empty strings or read out of bounds.

**Problem:** When calculating the common prefix of positive pathspecs, Git stripped that prefix from *all* pathspecs—including negative ones. For example, `git ls-files -- a/b/c a/b/d !a/b/` would strip `a/b/` from the negative pathspec, leaving an empty string that matches everything. The bug was introduced in 2013 (ef79b1f870) and affects both `git ls-files` and `git add`.

**Impact:** The patch adds a simple guard in `do_match_pathspec()` to skip prefix-stripping for negative pathspecs. The fix is minimal (11 lines in `dir.c`) and includes three new tests in `t/t6132-pathspec-exclude.sh` covering edge cases. The commit message clearly traces the bug’s origin and references a prior report by Thomas Haller.

**Today’s development:** Junio raised a substantive question about the patch’s design: whether the common prefix should instead be calculated across *both* positive and negative pathspecs, rather than skipping prefix-stripping for negative pathspecs entirely. For example, `git ls-files -- a/b/c a/b/d !a/b/` might be more intuitive if the prefix `a/b/` is stripped from all three pathspecs, leaving the negative pathspec relative to the full tree. The discussion remains open-ended, and the author will need to respond before the patch can proceed.

---

### OCSP staple validation: test harness decision pending
**What changed:** The `http.sslVerifyStatus` security feature patch, which adds OCSP staple validation to Git’s HTTPS certificate verification, is technically complete and ready for `next`. The only remaining question is whether to merge Patrick Steinhardt’s additional OCSP test harness (`t5585-http-ssl-ocsp.sh`), which exercises positive cases (valid staples) but adds complexity.

**Problem:** Git’s HTTPS certificate verification omits OCSP staple validation, leaving it vulnerable to accepting revoked certificates. The patch adds a boolean `http.sslVerifyStatus` option (default `false`) that enables staple validation via libcurl’s `CURLOPT_SSL_VERIFYSTATUS`, causing connections to fail if the server does not provide a staple. The feature is motivated by government customers (e.g., US Department of Defense PKI) who mandate OCSP stapling.

**Impact:** The patch touches `http.c`, `Documentation/config/http.adoc`, and test scripts. It is fail-closed for security (connection fails if staple missing) but defaults to `false` to avoid breaking remotes that do not support stapling. The test harness in question adds coverage for positive cases but remains unmerged due to concerns about complexity.

**Today’s development:** Junio asked Patrick Steinhardt which parts of the test harness feel uncertain or overly complex, noting the tests seem to cover essential scenarios but may not explicitly verify the case where the server does not provide a staple. The author (Grayson Gordon) framed the choice as a trade-off between thoroughness and succinctness, deferring to Junio’s preference. The thread is now awaiting Patrick’s response to resolve the final test-scope question before the patch graduates to `next`.

---

### `--[no-]range-diff-notes` series abandoned
**What changed:** Kristoffer Haugsbakk abandoned the `[PATCH 0/3] format-patch: add --[no-]range-diff-notes` series after Junio C Hamano raised a substantive usability concern about the option semantics.

**Problem:** The series aimed to add `--[no-]range-diff-notes` options to `git format-patch` to allow independent control over whether Git notes are included in the range-diff section versus the patches themselves. However, the current implementation did not support the intuitive use case where `--no-notes --range-diff-notes` suppresses notes in the patches but *includes* them in the range-diff.

**Impact:** The series touched `builtin/log.c`, `log-tree.c`, `revision.c`, `revision.h`, `Documentation/git-format-patch.txt`, and tests in `t/t3206-range-diff.sh`. It introduced a `rdiff_override_notes` flag in `struct rev_info` and renamed `rdiff_log_arg` to `rdiff_notes_arg`. The refactoring in patches 1 and 2 is now moot unless someone revives the effort with a cleaner design.

**Today’s development:** Junio confirmed the abandonment, restating the core usability issue: the interaction between `--range-diff-notes` and the existing `--notes` could not be defined in simple terms. He sketched a potential alternative design—where `--notes` controls only the final patch output and `--range-diff-notes` controls only the range-diff material—but noted no such redesign was proposed. The thread now ends without further action.

---

## In brief
- **ODB alternates refactoring:** Junio raised a security concern about patch 6/8 in Patrick Steinhardt’s series, which opens the `info/alternates` file in append mode (`"a"`) rather than using Git’s standard lockfile API. This could lead to partial writes or TOCTOU races during repository creation. The discussion may delay the series until the locking strategy is resolved.
- **Typo fixes:** Hardik Kumar posted a patch fixing two typos: "fractionnal" → "fractional" in `versioncmp.c` and "similiarity" → "similarity" in `t/t0022-crlf-rename.sh`. The changes are purely cosmetic and uncontroversial.
- **What’s cooking:** Junio’s "What’s cooking" report for August 2026 (#12) highlighted 567 non-merge commits in `master` since Git 2.55, with 76 topics in `next` and 215 in `seen`. Two topics graduated to `master`: a documentation fix for `git format-rev` and a heap-use-after-free fix in object name parsing. Notable stalled topics include `ec/commit-fixup-options` and `ps/libgit-in-subdir`.