# The Git Project -- Weekly Digest (2026/07/27 -- 2026/08/02)

## The period in brief

This was a **high-volume, high-impact week** on the Git mailing list, with **330 emails across 111 threads** covering seven active days. Traffic was **heavier than average** (roughly 47 emails/day), driven by **major feature series reaching maturity**, **substantive design debates**, and **a policy discussion about AI-assisted contributions**. The week’s standout developments: **`git branch --delete-merged` landed after 24 iterations**, **`git history squash` was ejected from `next` then revived**, and **`git replay --linearize` resolved its multi-branch ambiguity**. Two threads you absolutely should not miss: the **`git stash reword` data-loss risk** (blocked on a new reflog API) and the **AI-assisted contribution policy debate** sparked by a promisor-pack performance patch.

---

## Key developments

### `git branch --delete-merged` lands after 24-iteration journey
Harald Nordgren’s **seven-patch series** adding `--delete-merged` to `git branch` reached **implementation-complete status** in v24 and is now **queued for `next`**. The feature provides **safe automated cleanup** of local branches already merged into a specified commit, with **stacked-branch protection** (refusing to delete branches whose upstream is being deleted), **per-branch opt-out** (`branch.<name>.skipDeleteMerged`), and **repeatable arguments** (`--delete-merged origin/main --delete-merged origin/next`). The final iteration resolved two critical correctness issues: **accurate push-detection logic** in `branch_pushes_to_upstream()` and **consistent stacked-branch behavior** (clearing upstream configs for kept branches whose own upstream is being deleted). The series is **technically sound**, well-tested, and addresses a long-standing workflow pain point. Junio C Hamano has confirmed it is ready for merging pending final review.

**Key participants**: Harald Nordgren, Phillip Wood, Junio C Hamano.
**Files touched**: `builtin/branch.c`, `ref-filter.c`, `Documentation/git-branch.adoc`, `t/t3200-branch.sh`.
**Status**: Queued for `next`; expected to graduate unless integration issues surface.

---

### `git history squash` ejected from `next`, then revived with fixups
Toon Claes’s **`git history squash`** subcommand, part of the new `git history` suite, was **ejected from `next`** after Phillip Wood’s review exposed **fundamental flaws** in its reachability logic: the implementation incorrectly assumed `UNINTERESTING` commits imply a `BOTTOM` commit, failed to handle multi-tip histories, and had misleading UX around `--reedit-message`. The series was **revived in v11**, which made **message editing the default behavior** (resolving the UX debate) and added **stricter range validation** (rejecting ranges that reach a root commit or have more than one tip). The fixups address all correctness issues, and the series is now **functionally complete**. Junio’s prior "Will replace" sign-off on v7 and Matt Hunter’s explicit v10 sign-off signal intent to merge. The episode prompted Junio to **propose a temporary moratorium on new topics in `seen`** unless they receive substantive review, a direct response to the systemic review quality challenges highlighted by this series.

**Key participants**: Toon Claes, Phillip Wood, Junio C Hamano, Matt Hunter.
**Files touched**: `builtin/history.c`, `sequencer.c`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
**Status**: Ready for integration; queued for `next` in the next "What's cooking" cycle.

---

### `git replay --linearize` resolves multi-branch ambiguity
Toon Claes’s **`git replay --linearize`** series, which adds a `--linearize` option to flatten merge commits, **resolved its long-standing design debate** in v8 by **restricting `--linearize` to single-branch usage**. The restriction eliminates the "multi-branch ambiguity" (emergent reachability when concatenating multiple branches into a single linear chain) that had blocked the series in `next`. The v8 update aligns with Elijah Newren’s "minimal fix" proposal from v7 and preserves the option’s core functionality while deferring per-branch linearization (e.g., via `--ref` syntax) to future work. The series is now **technically complete** and addresses all prior feedback. Junio removed the previous version from `next` and will queue v8 for review. The only remaining open question is whether `--linearize` should eventually support per-branch linearization, but this is explicitly scoped as future work.

**Key participants**: Toon Claes, Elijah Newren, Junio C Hamano.
**Files touched**: `replay.c`, `replay.h`, `builtin/replay.c`, `Documentation/git-replay.adoc`, `t/t3650-replay-basics.sh`.
**Status**: Ready for review; queued for `next`.

---

### `git stash reword` blocked by data-loss risk and reflog API redesign
Emin Özata’s **`git stash reword`** feature, which allows users to edit stash messages after creation, hit a **major roadblock** when Junio C Hamano identified a **data-loss risk** in its implementation. The current approach slurps the entire reflog into memory, deletes it on-disk, and rewrites it with a single entry modified—creating a critical window where the reflog exists only in memory. If the process is killed after `refs_delete_reflog()` but before the transaction commits, the reflog is permanently lost. Junio proposed extending the reflog API to support **atomic replacement of individual entries** via `refs_reflog_edit_in_bulk()`, which would:
- Accept an array of edits (replace, insert, delete) targeting specific reflog entries by index.
- Handle index drift during insert/delete operations.
- Enforce stable sorting for consistent results across backends.
- Validate impossible sequences (e.g., deleting an entry before replacing it).
- Squash multi-line messages to a single line (matching `git stash push -m` behavior).

The discussion has shifted from "should we do this?" to "how should we do this?", treating the API extension as a **prerequisite for merging**. The patch remains **blocked until this work is done**, but the path forward is well-defined. The feature itself is **justified by concrete user workflows** (brian m. carlson, erik88) and is no longer controversial in principle.

**Key participants**: Emin Özata, Junio C Hamano, brian m. carlson, erik88.
**Files touched**: `builtin/stash.c`, `refs.c`.
**Status**: Blocked; API redesign (`refs_reflog_edit_in_bulk()`) required.

---

### `git add --resolved` sparks interface design debate
Junio C Hamano’s **four-patch series** introducing `git add --resolved` (staging only paths whose conflict markers have been removed) sparked a **philosophical debate** about Git’s interface conventions. The series consolidates duplicate conflict-marker detection logic, adds helpers for index removal with flags (`--dry-run`, `--verbose`), and includes a performance tweak for binary files. Michael Montalbo critiqued the **monolithic nature** of `--resolved`, arguing it conflates selection (unmerged paths) with policy (conflict-marker check). He proposed an alternative design: a general `--unmerged` selector with explicit policy flags like `--allow-conflict-markers` or `--skip-conflict-markers`. Junio defended `--resolved` as aligning with Git’s existing conventions (e.g., `git am --resolved`) and reflecting user intent, while Peff raised concerns about the feature potentially creating a **false sense of completeness** by staging only files with resolved conflict markers while ignoring related changes (e.g., updates to callers of a modified function). The debate remains unresolved, but the series is **technically complete** and ready for review. The next steps may involve **documentation tweaks** to clarify the feature’s scope or a warning in the command’s output.

**Key participants**: Junio C Hamano, Michael Montalbo, Jeff King (Peff).
**Files touched**: `merge-ll.c`, `read-cache.c`, `builtin/add.c`, `Documentation/git-add.txt`, `t/t2207-add-resolved.sh`.
**Status**: Under review; interface design debate ongoing.

---

### AI-assisted contribution policy debate erupts
Arijit Banerjee’s **performance patch** for `git index-pack` (speeding up promisor packs by 15–26% via thread-local oidmaps) became the focal point of a **policy debate** about AI-assisted contributions. The commit message disclosed AI assistance, prompting **brian m. carlson** to object on policy grounds, citing `SubmittingPatches`’ requirement that authors certify full understanding of non-trivial code. Junio clarified that Git’s stance is not an outright ban but a **cautious, case-by-case approach**, and suggested LLVM’s rejection of "extractive contributions" as a potential model. The thread stalled on whether the performance benefit justifies an exception, with no maintainer yet weighing in on the trade-off. Arijit’s follow-up brainstormed procedural ideas (karma systems, experimental release trains) but did not advance the core question. The debate highlights a **growing tension** between Git’s traditional review standards and the increasing use of AI tools in development.

**Key participants**: Arijit Banerjee, brian m. carlson, Junio C Hamano.
**Files touched**: `builtin/index-pack.c`, `oidmap.c`, `oidmap.h`.
**Status**: Stalled; awaiting maintainer guidance on policy.

---

### `git repack --drop-filtered` for partial clones refines safety guards
Siddharth Shrimali’s **RFC v2 series** introducing `--drop-filtered` to `git repack` (allowing users to reclaim disk space in partial clones by removing locally cached promisor blobs exceeding a size threshold) saw a **design pivot** in response to Junio’s feedback. The current implementation includes **safety guards** (merge/rebase/cherry-pick checks and index validation) to prevent culling objects that might be needed for in-progress operations. Junio proposed replacing these checks with a **time-based heuristic** (avoid culling objects fetched within the last 20 minutes), which Siddharth endorsed. The discussion now centers on whether to implement this heuristic in the next revision or defer it to a follow-up. The core functionality (enumeration, repacking, implied `-d`) remains unchanged. The series is **well-structured** and addresses a real pain point for partial clone users, but the safety guard design remains a key open question.

**Key participants**: Siddharth Shrimali, Junio C Hamano.
**Files touched**: `builtin/repack.c`, `repack-filtered.c`, `repack-promisor.c`, `Documentation/git-repack.adoc`, `t/t7706-repack-drop-filtered.sh`.
**Status**: Under review; safety guard design debate ongoing.

---

### Git 3.0: lowercase-only hex object IDs sparks philosophical debate
brian m. carlson’s **RFC series** proposing a Git 3.0 breaking change to enforce lowercase-only hex object IDs continued to spark **philosophical debate** about the Robustness Principle. Junio invoked the principle to argue that Git should remain liberal in what it accepts, even in a breaking-change release, while brian countered by citing modern security practices (TLS, HTTP request smuggling) to argue that lenient parsing is now a security anti-pattern. Jeff King (Peff) flagged an **edge case**: uppercase hex in object payloads (e.g., commits) creates parallel histories, as Git does not normalize case when hashing contents. Brian acknowledged this as a real issue complicating hash-algorithm compatibility (such objects cannot be round-tripped through Git’s interoperability code). The thread remains focused on the philosophical divide, with no maintainer decision yet. The first five infrastructure patches are uncontroversial, but the sixth (enforcement) hinges on resolving the broader justification debate.

**Key participants**: brian m. carlson, Junio C Hamano, Jeff King (Peff).
**Files touched**: `hex.c`, `cache.h`, `Documentation/RelNotes/3.0.0.txt`.
**Status**: Under discussion; philosophical debate ongoing.

---

## In brief

**`git rebase --update-refs` symref bugfix** -- Son Luong Ngoc’s **2-patch series** fixing `git rebase --update-refs` behavior with symrefs **resolved its design debate** after Phillip Wood provided concrete evidence (TopGit’s use of HEAD symrefs pointing to `refs/top-bases/...`) that the edge case is actively relied upon. The series is **ready for merging** after addressing minor refinements (explanatory comments, test organization). **Files**: `sequencer.c`, `branch.c`. **Status**: Ready for `next`.

**`git cat-file --batch-command` assertion fix** -- Jeff King (Peff) submitted a **minimal fix** for a long-standing crash in `git cat-file --batch-command` when the `contents` command is issued without `%(objecttype)` in the format string. The patch saves and restores `data->info.typep` around the `contents` command, avoiding the NULL dereference while preserving performance for `info` commands. The fix includes a new test in `t1006-cat-file.sh` and is **queued for `next`**. **Files**: `builtin/cat-file.c`, `t/t1006-cat-file.sh`. **Status**: Queued for `next`.

**Pack-bitmap off-by-one edge case fixed** -- David Lin’s **bugfix** for the pack-bitmap subsystem, where objects at position zero in the base bitmap were incorrectly excluded from being recognized as already represented, is **accepted for integration**. The fix changes a conditional check from `pos > 0` to `pos >= 0` in `find_objects()`, eliminating redundant work. The patch includes expanded test coverage for both pseudo-merge and non-pseudo-merge scenarios. **Files**: `pack-bitmap.c`, `t/t5333-pseudo-merge-bitmaps.sh`. **Status**: Accepted.

**Security fix for Git for Windows (CVE-2026-32631)** -- Johannes Schindelin posted a **security fix** for Git for Windows preventing credential exfiltration when cloning a crafted repository with symlinks enabled. The vulnerability involves symlink type auto-detection triggering an SMB connection to a UNC path or drive-less absolute path (e.g., `\attacker\share`), exposing the user’s NTLMv2 hash. The fix modifies `compat/mingw.c` to skip auto-detection for symlink targets starting with a backslash, defaulting to file symlinks and warning users to set `symlink=dir` in gitattributes if needed. The patch is already released in Git for Windows v2.53.0(3) and is **queued for integration**. **Files**: `compat/mingw.c`. **Status**: Queued for integration.

**`git mv` symlink-handling fix** -- Lucas Zamboni Orioli’s **two-patch series** fixing misleading errors in `git mv` (when the destination’s parent directory does not exist or contains problematic components) and adding an early check to reject tracked symlinks in the destination path received **final maintainer approval**. The v5 update extracted shared logic into a `needs_worktree_rename()` helper function, ensuring consistency between the early check and the actual rename operation. **Files**: `builtin/mv.c`, `t/t7001-mv.sh`. **Status**: Ready for `next`.

**`git bisect --reset-when-found`** -- Harald Nordgren’s **series** adding an auto-reset option to `git bisect` reached a polished v6 after Junio’s final clean-ups (reading `refs/bisect/bad` directly before reset, using a sentinel value to remove a boolean flag). The series is **ready for `next`**, with all surface-level feedback addressed. **Files**: `builtin/bisect.c`, `bisect.h`, `Documentation/git-bisect.txt`, `t/t6030-bisect-porcelain.sh`. **Status**: Ready for `next`.

**`git config --global` documentation vs. behavior discrepancy** -- Nils Fahldieck and Ben Knoble reinforced the argument for aligning `git config --global`’s behavior with its documentation by making it read both `~/.gitconfig` and the XDG config file, matching `--get`’s behavior. The maintainer’s preference for documenting the single-file behavior remains unchallenged, but the cross-command inconsistency (`--get` vs. `--global`) and unintuitive behavior with `--global --no-includes` strengthen the case for a code change. **Status**: Awaiting contributor follow-up.

**`git stash push` with paths stashes unrelated files** -- Yuri reported that `git stash push` with explicit paths unexpectedly stashes changes from an unrelated directory, including an untracked file not listed on the command line. The report is reproducible in the FreeBSD ports tree and suggests a bug in `git stash`’s path-filtering logic, possibly involving untracked files or symlinks. **Status**: No patch or fix proposed yet.

**GSoC `git cat-file --batch-command` `%(objecttype)` support** -- Pablo Sabater’s **GSoC series** extending `git cat-file --batch-command` to support the `%(objecttype)` placeholder in remote-object-info queries is **complete at v2**. The six-patch series addresses all review feedback, including patch reordering, test portability fixes, and documentation updates. Junio’s review of the first patch confirmed the dynamic request logic is safe, and the series is **ready for maintainer consideration**. **Files**: `builtin/cat-file.c`, `Documentation/git-cat-file.txt`, `t/t1006-cat-file.sh`. **Status**: Ready for review.

**`git add -e` refactoring** -- Gatla Vishwahwar Reddy’s **patch** replacing a subprocess call to `git apply` with internal API in `git add -e` is **approved for `next`**. The refactoring is mechanical and uncontroversial. **Files**: `builtin/add.c`. **Status**: Queued for `next`.

**`git index-pack` race-condition fix** -- Ted Nyman’s **v6 packfile URI race-condition fix** received **final approval from Peff**, clearing the last technical hurdle before merge. The series is **fully approved** by all key reviewers (Junio, Taylor Blau, Peff) and in a "ready for merge" state. **Files**: `http.c`, `fetch-pack.c`, `t/t5550-http-fetch-dumb.sh`, `t/t5702-protocol-v2.sh`. **Status**: Ready for merge.

**`git maintenance` regression fix** -- David Lin’s **patch** fixing a regression in `git maintenance`’s "none" strategy handling (broken since October 2025) received surface-level review from Junio, who requested an in-body `From:` line to align author and sign-off addresses. **Files**: `builtin/gc.c`. **Status**: Under review.

**`git merge-base --is-ancestor` tests** -- Nikolaus Schuetz’s **test coverage** for `git merge-base --is-ancestor` is **queued for integration**. The v2 revision reuses the existing test repository history and adds a test case for rejecting excess arguments. **Files**: `t/t6010-merge-base.sh`. **Status**: Queued for integration.

**`test_commit_body()` helper** -- Shlok Kulshreshtha’s **series** introducing a `commit_body()` helper to replace 61 duplicated `git cat-file commit ... | sed ...` patterns in the test suite is **queued for `next`**. The helper is a clean refactoring with no user-visible impact. **Files**: `t/test-lib-functions.sh`, `t/t0000-basic.sh`. **Status**: Queued for `next`.

**`git replay` config documentation** -- Kristoffer Haugsbakk’s **series** synchronizing `replay.refAction` documentation between `git-config(1)` and `git-replay(1)` is **cooking in `next`**. The series implements Git documentation conventions through conditional linking, description lists, and consistent default value placement. **Files**: `Documentation/git-config.txt`, `Documentation/git-replay.txt`. **Status**: Cooking in `next`.

**`git-interpret-trailers` documentation overhaul** -- Kristoffer Haugsbakk’s **11-patch series** overhauling the `git-interpret-trailers` man page to clarify terminology, explain the strict key format, and document how comment lines are handled is **cooking in `next`**. The series addresses a prior bug report about silent failures when parsing keys containing periods. **Files**: `Documentation/git-interpret-trailers.txt`, `trailer.c`, `t/t7513-interpret-trailers.sh`. **Status**: Cooking in `next`.

**`git refs` documentation cleanup** -- Kristoffer Haugsbakk’s **two-patch series** elevating a ref-migration warning into a prominent admonition block and adding a cross-reference to `git-maintenance` is **queued for `seen`**. The changes are purely presentational. **Files**: `Documentation/git-refs.txt`, `Documentation/git-maintenance.txt`. **Status**: Queued for `seen`.

**Sparse-index segfault fix** -- Derrick Stolee’s **minimal fix** for a segfault when collapsing a full index to a sparse index in the presence of an intent-to-add entry outside the sparse-checkout cone is **confirmed for `next`**. The patch adds a guard clause in `convert_to_sparse_rec()` to skip cache-tree nodes with negative `entry_count`. **Files**: `sparse-index.c`, `t/t1092-sparse-checkout-compatibility.sh`, `t/t3705-add-sparse-checkout.sh`. **Status**: Cooking in `next`.

**`git merge` memory leak** -- Toon Claes’s **patch** plugging a long-standing memory leak in `git merge` when `merge.defaultToUpstream` is enabled is **queued for `seen`**. **Files**: `builtin/merge.c`, `t/t7600-merge.sh`. **Status**: Queued for `seen`.

**`git worktree add` bugfix** -- Matthias Aßhauer’s **series** fixing an out-of-bounds read in `git worktree add` saw progress: René Scharfe clarified that the validation patch (rejecting empty strings with a reference) is necessary for the `"" HEAD` edge case. The pointer-underflow fix in `worktree_basename()` remains the more urgent change. **Files**: `worktree.c`, `t/t2400-worktree-add.sh`. **Status**: Under review.

**`git blame` performance regression fix** -- Toon Claes’s **patch** fixing a performance regression in `git blame` (introduced by the ODB abstraction rework) is **under review**. The fix restores the documented behavior of `git maintenance`’s "none" strategy. **Files**: `builtin/blame.c`. **Status**: Under review.

**`git config` synopsis-style conversion** -- Jean-Noël Avila’s **ongoing effort** to convert all Git man pages to a consistent AsciiDoc synopsis format produced a **documentation patch** for `git config`. The patch is part of a larger, long-running series. **Files**: `Documentation/git-config.txt`. **Status**: Under review.

**`git grep` macOS regex leak workaround** -- Chungmin Lee’s **patch** preventing gigabyte leaks in `git grep` over binary files on macOS is **ready for merge**. The workaround avoids the leak by skipping regex compilation for binary files. **Files**: `grep.c`, `t/t7816-grep-binary-pattern.sh`. **Status**: Ready for merge.

**`git merge-base --is-ancestor` test coverage** -- Nikolaus Schuetz’s **test patch** for `git merge-base --is-ancestor` is **queued for integration**. The v2 revision reuses the existing test repository history. **Files**: `t/t6010-merge-base.sh`. **Status**: Queued for integration.

**`git stash` time-based selector retraction** -- Junio C Hamano **retracted** his two-patch series proposing to reject time-based stash selectors in `git stash drop` and `git stash pop` after discovering the reflog expiration machinery already limits removal to a single entry. **Files**: None. **Status**: Retracted.

**`--date=elapsed` for Git** -- Israel Roldan proposed a new `--date=elapsed` (and `--date=elapsed-pad`) date format for Git, along with corresponding `%aX` and `%cX` pretty-format placeholders. The feature provides a precise, script-friendly elapsed-time format (e.g., "1y 11M 5d 13h 5m 13s") modeled after `uptime(1)`. **Files**: `date.c`, `Documentation/pretty-formats.txt`, `t/t4202-log.sh`. **Status**: Under discussion.

**`git filter-branch` bugfix** -- Grant Moyer fixed a **long-standing inversion** in commit mapping logic with `--state-branch`, causing the map directory to be populated backwards. The patch is **self-contained** and includes a test. **Files**: `git-filter-branch.sh`, `t/t7003-filter-branch.sh`. **Status**: Ready for review.

**`git interactive` feature request** -- A new `git interactive` command (terminal menu for common operations) drew skepticism from Michael Montalbo, who questioned its necessity given existing tools (`git-gui`, `gitk`). The author has not yet addressed the overlap or provided a stronger rationale. **Status**: Under discussion.

**Monorepo remote helper** -- Zhonghua Zhu proposed `blackgit`, a proof-of-concept remote helper to optimize monorepo workflows (Perforce-style narrow clone). The idea is exploratory, with no patches or benchmarks yet. **Status**: Under discussion.

**Sparse-checkout performance fix** -- Sahitya Chandra’s **patch** to avoid index expansion in `repo_index_has_changes()` for unborn HEAD (using the empty tree) is a targeted win for sparse-checkout/sparse-index. The change is self-contained and well-tested. **Files**: `repository.c`, `t/t1092-sparse-checkout-compatibility.sh`. **Status**: Under review.

---

## Looking ahead

### Topics likely to dominate next week

- **`git stash reword` reflog API redesign** -- The `refs_reflog_edit_in_bulk()` API proposed by Junio C Hamano is a **significant new addition** to Git’s internals. Track this for potential unification with other reflog operations (e.g., `stash drop`, `reflog expire`). The API’s design (edit array, index drift handling, stable sorting) will likely see further refinement.
- **`git repack --drop-filtered` safety guards** -- Siddharth Shrimali’s **time-based heuristic** (avoid culling objects fetched within the last 20 minutes) is a key open question. The next revision may implement this heuristic or defer it to a follow-up.
- **AI-assisted contribution policy** -- The debate sparked by Arijit Banerjee’s promisor-pack performance patch is **likely to continue**, with potential maintainer guidance or a formal policy update. The discussion may expand to include procedural ideas (karma systems, experimental release trains) or a broader review of `SubmittingPatches` guidelines.
- **Git 3.0: lowercase-only hex object IDs** -- The philosophical debate between brian m. carlson and Junio C Hamano (security strictness vs. Robustness Principle) may require **PLC input** to resolve. The first five infrastructure patches are uncontroversial, but the sixth (enforcement) hinges on this debate.
- **`git add --resolved` interface design** -- The debate between Junio’s monolithic `--resolved` option and Michael Montalbo’s modular `--unmerged` selector with policy flags may prompt **further discussion** or a maintainer decision. Documentation tweaks or a warning in the command’s output are possible next steps.

### Ongoing efforts making progress

- **ODB abstraction** -- Patrick Steinhardt’s **ODB abstraction series** continues to receive substantive review, with Toon Claes elevating the first patch’s importance from a mechanical cleanup to a **latent correctness bug** affecting repositories with alternates and compatibility hash algorithms. The series is **making steady progress** but remains a long-term effort.
- **`git replay --linearize` per-branch support** -- Toon Claes’s `--ref` proposal for per-branch linearization remains a **potential future direction** for the `git replay --linearize` feature. The current series is complete, but this enhancement could reappear in a follow-up.
- **`diff-provider` interface** -- Michael Montalbo’s **RFC v7** proposing a unified abstraction layer for diff hunks is **technically complete for its scope** and awaiting design feedback. If accepted, the work will return as separate series: (1) interface and `diff-hunks` store, (2) external process provider, (3) content enrichment.
- **`git bisect --reset-when-found`** -- Harald Nordgren’s **series** is **ready for `next`** and expected to graduate unless integration issues surface. The feature is a clean addition with no known technical concerns.