# Git mailing list daily digest for 2026/09/03

## The day in brief
The Git project saw significant progress on several fronts today. The `--missing-only` option for `git rev-list` was clarified and approved for `next`, while a critical usability concern was raised for the `--drop-filtered` option in `git repack`. Multiple bugfixes advanced, including fixes for memory leaks, race conditions, and platform-specific issues. The Outreachy December 2026 cohort gained momentum with additional mentor volunteers, and several documentation improvements were proposed.

## Notable threads

### `git rev-list --missing-only` approved for `next`
**What changed?**
The `--missing-only` option for `git rev-list` was clarified to support GitLab's Gitaly partial clone workflows. The feature enables output of only missing object IDs without post-processing, addressing GitLab's need to efficiently identify objects not present in a partial clone during transaction packing.

**Why it matters**
This feature provides a script-friendly way to list only missing objects while preserving existing `--missing=` formatting options. It's particularly useful for GitLab's Gitaly workflow, where missing objects are recorded as dependencies in a single-pass transaction packing operation.

**Key technical details**
- Files: `builtin/rev-list.c`, `Documentation/rev-list-options.adoc`, `t/t6222-rev-list-missing.sh`
- New flag: `--missing-only` filters output to missing objects only
- Requires `--missing=print` or `--missing=print-info`
- Rejects incompatible options (`--count`, `--disk-usage`) with clear error messages
- Output format: one OID per line (no `?` prefix), or `path=`/`type=` fields if `--missing=print-info` is used

**Current status**
Approved and queued for `next`. The commit message was strengthened to clarify the Gitaly workflow motivation.

---

### Usability concern raised for `git repack --drop-filtered`
**What changed?**
Samuel Bronson reported a usability pain point with the `--drop-filtered` option for `git repack`. The current implementation dies when encountering an index-referenced blob, forcing users to restart the entire (potentially slow) enumeration process.

**Why it matters**
The `--drop-filtered` option is designed to safely reclaim disk space in partial clones by removing locally cached promisor blobs exceeding a user-specified size threshold. However, the current fail-fast behavior creates a poor user experience, particularly for large repositories.

**Key technical details**
- Files: `builtin/repack.c`, `repack-filtered.c`
- New CLI option: `--drop-filtered` (real runs) and `--dry-run` (reporting only)
- Current behavior: fatal error when encountering index-referenced blobs
- Proposed change: skip index-referenced blobs with a warning instead of failing

**Current status**
Proposed for `next` but may require a v6 to address this feedback. The author must decide whether to adopt the warning-based approach or defend the current fail-fast behavior.

---

### `git checkout -m` autostash conflict handling refined
**What changed?**
A two-patch series refining `git checkout -m` autostash conflict handling was approved for `next`. The series reserves exit code 1 for conflicts and visually separates autostash conflict advice from the branch-switch confirmation message.

**Why it matters**
This change improves the user experience during conflict resolution by making error messages clearer and more consistent with Git's established conventions.

**Key technical details**
- Files: `sequencer.c`, `builtin/checkout.c`, `stash.h`, `builtin/stash.c`, `Documentation/git-stash.adoc`
- New `enum stash_apply_result` with values: `STASH_APPLY_CLEAN`, `STASH_APPLY_CONFLICT`, `STASH_APPLY_ERROR`
- Exit code 1 for conflicts, 128 for other errors
- Blank line separates conflict advice from branch-switch message

**Current status**
Approved and queued for `next`. Both patches are "good to me" according to Junio C Hamano.

---

### `git maintenance` `rerere gc` task improved
**What changed?**
A two-patch series improving the heuristic for the `git maintenance` `rerere gc` task was posted. The series replaces the hard-coded 60-day cutoff with a dynamic heuristic that estimates stale entries and triggers the task only when the estimate meets or exceeds a configurable threshold.

**Why it matters**
The geometric maintenance strategy's aggressiveness was causing race conditions with in-progress rebases. This change makes the maintenance task more efficient and less likely to interfere with user operations.

**Key technical details**
- Files: `builtin/gc.c`, `rerere.c`
- New functions: `rerere_gc_cutoffs()`, `rerere_id_is_stale()`, `rerere_gc_estimate()`
- Dynamic heuristic samples the `rr-cache` directory to count stale entries
- Configurable threshold (default: 512)

**Current status**
Under review. Derrick Stolee provided a "Looks Good To Me" (LGTM) for both patches.

---

### Outreachy December 2026 cohort gains momentum
**What changed?**
Pablo Sabater confirmed his availability to co-mentor an Outreachy project, joining the growing list of volunteers.

**Why it matters**
Git's participation in Outreachy provides valuable opportunities for new contributors from underrepresented groups. The program has a proven track record of success, with last year's intern co-mentoring a GSoC intern and presenting at the upcoming Git conference.

**Current status**
The thread now has four confirmed or potential mentors and two org admin candidates. The deadline for mentoring organizations to sign up is September 11, 2026.

## In brief

- **[PATCH] imap-send: add --draft option**: Adds `--draft` option to `git imap-send` to mark uploaded messages as drafts. Under review; concerns raised about libcurl version requirement.
- **[PATCH] builtin/history: guard against NULL tree**: Prevents a NULL-pointer dereference crash in `git history split`. Approved for `next`.
- **[PATCH] dir: fix common prefix calculation with leading exclude pathspec**: Fixes a long-standing edge case in `common_prefix_len()` where an exclude pathspec appearing first in the list causes incorrect common prefix calculation. Queued for merging.
- **[PATCH] push: fix uninitialized variable in `--force-if-includes`**: Fixes an uninitialized variable bug in the `--force-if-includes` safety mechanism. v2 posted addressing review feedback.
- **[PATCH] git-gui: fix commit with long messages on Windows**: Fixes a Windows-specific pipe-handling issue in Git GUI. Under review; reproducibility concerns raised.
- **Memory leak in `git history reword`**: Reported a memory leak in `git history reword --dry-run` introduced by the merge of `ps/receive-pack-shallow-optim`. Under investigation.
- **Inconsistent branch name validation in porcelain commands**: Discussion continues about whether to reject bare `@` as a branch name. No consensus yet.
- **[PATCH] contrib/git-contacts: ignore boundary commits in blame output**: Fixes misclassification of boundary commits as legitimate contributors. Under review.
- **[PATCH] pack-objects: fix uninitialized variable in partial clones**: Fixes a logic error causing inefficient fetching of missing objects in partial clones. Under review.
- **[PATCH] Documentation: clarify partial clone dynamic object fetching**: Corrects inaccuracies in `Documentation/technical/partial-clone.adoc`. Approved by Elijah Newren.