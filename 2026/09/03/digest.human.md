# Git mailing list daily digest for 2026/09/03

## The day in brief
The Git project saw significant progress on several fronts today. A long-awaited `--missing-only` option for `git rev-list` was approved and queued for `next`, addressing GitLab's Gitaly workflow needs. The `git repack --drop-filtered` series received substantive usability feedback that may require a v6 revision. Meanwhile, the Outreachy December 2026 cohort gained another mentor, strengthening Git's application. Several bugfixes and documentation improvements also advanced, including fixes for memory leaks, pathspec handling, and partial clone documentation.

## Notable threads

### `git rev-list --missing-only` approved and queued for `next`
**What changed**: The `--missing-only` option for `git rev-list` has been approved and queued for `next`. This feature enables output of only missing object IDs without post-processing, addressing GitLab's need to efficiently identify objects not present in a partial clone during transaction packing.

**Why it matters**: This feature provides a script-friendly way to list only missing objects while preserving existing `--missing=` formatting options. It's particularly useful for GitLab's Gitaly workflow, which uses a one-shot walk during transaction packing to record missing objects as dependencies.

### Key technical details

- Files: `builtin/rev-list.c`, `Documentation/rev-list-options.adoc`, `t/t6022-rev-list-missing.sh`
- New flag: `--missing-only` (filters output to missing objects only)
- Requires `--missing=print` or `--missing=print-info`
- Rejects incompatible options (`--count`, `--disk-usage`) with clear error messages
- Output format: one OID per line (no `?` prefix), or `path=`/`type=` fields if `--missing=print-info` is used

**Current status**: Queued for `next`. Junio C Hamano has approved the feature after the commit message was strengthened to clarify the Gitaly workflow.

[2026/09/03/20-45-50 by Siddharth Asthana]

---

### `git repack --drop-filtered` usability feedback may require v6
**What changed**: Samuel Bronson provided substantive usability feedback on the `git repack --drop-filtered` series, identifying a pain point where the current implementation dies when encountering an index-referenced blob, forcing users to restart the entire enumeration process.

**Why it matters**: The series aims to safely reclaim disk space in partial clones by removing locally cached promisor blobs exceeding a user-specified size threshold. However, the current fail-fast behavior creates a poor user experience for large repositories.

### Key technical details

- Proposed change: Skip index-referenced blobs with a warning instead of failing
- Attached patch demonstrates the change but omits test updates
- Affects patch 6/6 (safety guards)
- The change would improve usability without compromising safety

**Current status**: Proposed for `next` but may need a v6 to address this feedback before integration.

[2026/09/03/21-52-06 by Samuel Bronson]

---

### Outreachy December 2026 cohort gains another mentor
**What changed**: Pablo Sabater confirmed his availability to co-mentor an Outreachy project for the December 2026 cohort, joining the growing list of volunteers.

**Why it matters**: Git's participation in Outreachy depends on having sufficient mentors and org admins. With four confirmed or potential mentors and two org admin candidates, the project is well-positioned to submit a strong application.

**Current status**: The thread now has four confirmed or potential mentors and two org admin candidates. The deadline for mentoring organizations to sign up is September 11, 2026.

[2026/09/03/10-24-48 by Pablo Sabater]

---

### `git rerere` lock race fix progresses
**What changed**: The fix for a race condition between `git rerere` and background maintenance during rebase has made significant progress, with the design now settled for a two-patch series.

**Why it matters**: The race condition causes rebases to fail when background maintenance holds the `MERGE_RR.lock`, creating a poor user experience. The fix addresses this by making `rerere_setup()` return errors instead of dying and disabling background maintenance during rebase.

### Key technical details

- Two-patch series: locking changes and config overrides
- `rerere_setup()` will wait up to 1 second for non-maintenance callers
- Background maintenance will skip the lock entirely (timeout=0)
- Uses `LOCK_REPORT_ON_ERROR` for better error reporting
- Merge and apply backends will behave consistently

**Current status**: Design settled; awaiting Phillip Wood's review of the v2 implementation.

[2026/09/03/08-11-05 by Thomas Bachem]

---

### Pathspec handling bugfix series queued for merging
**What changed**: A bugfix series addressing a long-standing edge case in `common_prefix_len()` and a memory-safety issue in `match_pathspec_with_flags()` has been queued for merging.

**Why it matters**: The series fixes an edge case where an exclude pathspec appearing first in the list causes `common_prefix_len()` to incorrectly return zero, even when subsequent positive pathspecs share a directory. It also prevents a heap-buffer-overflow when negative pathspecs are shorter than the common prefix derived from positive pathspecs.

### Key technical details

- Three-patch series: preparatory refactoring, memory-safety fix, core fix
- Files: `dir.c`, `pathspec.c`, `pathspec.h`, `t/t6132-pathspec-exclude.sh`, `t/unit-tests/u-dir.c`
- New behavior: `common_prefix_len()` correctly returns the common prefix of positive pathspecs even when an exclude pathspec appears first
- Memory-safety fix: negative pathspecs are matched with a zero prefix to prevent out-of-bounds reads

**Current status**: Queued for merging. Junio C Hamano has confirmed the integration plan.

[2026/09/03/18-51-42 by Junio C Hamano]

---

## In brief
- **`git checkout -m` autostash conflict handling**: The v5 series refining autostash conflict handling has been queued for `next` after all prior review feedback was addressed. [2026/09/03/18-53-27 by Junio C Hamano]
- **`git var` extension**: Junio C Hamano provided a substantive review of the `git var` extension patch, requesting documentation improvements, code structure changes, and usability edge case handling. [2026/09/03/17-40-48 by Junio C Hamano]
- **`git push` shallow-clone performance**: Elijah Newren corrected Stolee's over-exclusion critique, clarifying that the flaw is a false-positive failure (rejecting valid refs unnecessarily) due to `receive-pack`'s connectivity checks, not a corruption risk. [2026/09/03/09-22-39 by Elijah Newren]
- **`git imap-send --draft`**: The v2 patch for the `--draft` option addresses Junio's requests to remove the `Assisted-by: LLM` trailer and improve test cleanup placement. [2026/09/03/20-00-15 by Aleksei Sviridkin]
- **`git history` NULL-dereference fix**: Patrick Steinhardt confirmed the fix "looks good to me," marking it ready for integration. [2026/09/03/07-52-18 by Patrick Steinhardt]
- **`git-contacts` boundary commit fix**: A bugfix for the `git-contacts` contrib script prevents false positives in reviewer suggestions by ignoring boundary commits in blame output. [2026/09/03/12-55-27 by Aleksei Sviridkin]
- **`git cherry-pick --no-commit` documentation**: The documentation patch clarifies that `CHERRY_PICK_HEAD` is not created when a `--no-commit` cherry-pick fails due to conflicts, addressing a long-standing asymmetry with `git revert`. [2026/09/03/21-45-53 by Aleksei Sviridkin]
- **Partial clone documentation**: The v2 patch narrows the scope of the "one-at-a-time" fetch limitation in `Documentation/technical/partial-clone.adoc` to the fallback path in `odb.c`. [2026/09/03/20-00-13 by Aleksei Sviridkin]
- **`git history reword` memory leak**: A memory leak in `git history reword --dry-run` was reported, introduced by the merge of `ps/receive-pack-shallow-optim`. [2026/09/03/15-26-44 by Kristoffer Haugsbakk]
- **Git GUI Windows fix**: Johannes Sixt reported being unable to reproduce the Windows-specific pipe-handling issue in Git GUI and questioned the necessity of the fix. [2026/09/03/17-49-40 by Johannes Sixt]
- **`git maintenance` rerere gc heuristic**: Patrick Steinhardt posted a two-patch series improving the heuristic for the `git maintenance` `rerere gc` task, replacing the hard-coded 60-day cutoff with a dynamic heuristic. [2026/09/03/09-04-56 by Patrick Steinhardt]