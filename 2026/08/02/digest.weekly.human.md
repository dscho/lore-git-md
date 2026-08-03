# The Git Project -- Weekly Digest for 2026/07/27 -- 2026/08/02

## The period in brief

This was an **active and consequential week** on the Git mailing list, with **369 emails across 111 threads** over six days. Traffic was **heavy and technically dense**, dominated by **design debates, regression fixes, and long-running feature series** reaching key milestones. The standout developments: **`git replay --linearize` resolved its multi-branch ambiguity**, **`git history squash` was ejected from `next` after fundamental flaws were exposed**, and **`git branch --delete-merged` reached final form** after 24 iterations. Two security-related discussions—**Git for Windows credential exfiltration (CVE-2026-32631)** and **brian m. carlson’s lowercase-hex RFC**—highlighted the project’s ongoing attention to correctness and security. The week also saw **substantive progress on promisor-pack performance**, **ODB abstraction follow-ups**, and **new RFCs** proposing architectural changes to diff and repack machinery.

---

## Key developments

### `git replay --linearize` resolves multi-branch ambiguity, unblocking integration
The `git replay --linearize` series, which adds a `--linearize` option to flatten merge commits, **resolved its long-standing design impasse** over how to handle multiple branches. Toon Claes’s **v8** restricts `--linearize` to a single positive ref, eliminating the "multi-branch ambiguity" that had blocked the series in `next`. This aligns with Elijah Newren’s "minimal fix" proposal and preserves the option’s core functionality while avoiding emergent reachability issues. The series is now **technically complete**, with Junio removing the prior version from `next` and queuing v8 for review. The only remaining open question is whether `--linearize` should eventually support per-branch linearization (e.g., via `--ref` syntax), but this is deferred for future work.

**Key participants**: Toon Claes, Elijah Newren, Junio C Hamano.
**Status**: Ready for `next`; queued for integration.

---

### `git history squash` ejected from `next` after fundamental flaws exposed
Toon Claes’s `git history squash` subcommand, part of the new `git history` suite, was **ejected from `next`** after Phillip Wood’s review exposed **critical correctness flaws** in its reachability logic. The implementation incorrectly assumed `UNINTERESTING` commits imply a `BOTTOM` commit or preclude root commits, and failed to handle multi-tip histories (e.g., `git history squash ^A C D` in `-A-B-C \ D`). The `--reedit-message` option was also misleading, as it created a new commit rather than re-editing an existing one. Phillip is developing fixups to address the reachability logic, but the series may require **significant rework**. Junio’s ejection signals **heightened scrutiny for future `next` topics**, and he has proposed a **temporary moratorium on new `seen` topics** unless they receive substantive review.

**Key participants**: Toon Claes, Phillip Wood, Junio C Hamano.
**Status**: Ejected from `next`; fixups in progress.

---

### `git branch --delete-merged` reaches final form (v24)
Harald Nordgren’s **24-iteration series** adding `--delete-merged` to `git branch` is now **implementation-complete and ready for merging**. The series introduces safe automated local branch cleanup with comprehensive filtering and safety controls. The v24 update fixes two critical issues identified by Phillip Wood: (1) **corrected push-detection logic** in `branch_pushes_to_upstream()` to accurately determine whether pushing a branch would update its upstream, and (2) **simplified stacked-branch protection** that clears upstream configs for kept (merged) branches whose own upstream is being deleted. The implementation supports repeatable `--delete-merged` arguments (e.g., `--delete-merged origin/main --delete-merged origin/next`) and optional positional patterns (e.g., `topic-*`). Junio has confirmed the series is ready for merging pending final review.

**Key participants**: Harald Nordgren, Phillip Wood, Junio C Hamano.
**Status**: Ready for merge; queued for `next`.

---

### `git stash reword` blocked by reflog API redesign
Emin Özata’s `git stash reword` feature, which allows users to edit stash messages after creation, is **blocked by a fundamental correctness issue**: the current implementation slurps the entire reflog into memory, deletes it on-disk, and rewrites it, creating a critical window where the reflog exists only in memory. If the process is killed after `refs_delete_reflog()` but before the transaction commits, the reflog is permanently lost. Junio proposed extending the reflog API to support **atomic replacement of individual entries** via `refs_reflog_edit_in_bulk()`, which would eliminate the risk and make the operation truly atomic. The discussion has shifted from "should we do this?" to "how should we do this?", treating the API extension as a prerequisite for merging. The patch remains **blocked until this work is done**.

**Key participants**: Emin Özata, Junio C Hamano.
**Status**: Blocked; API redesign required.

---

### Security fix for Git for Windows (CVE-2026-32631)
Johannes Schindelin posted a **security fix** for Git for Windows that prevents credential exfiltration when cloning a crafted repository with symlinks enabled. The vulnerability, assigned **CVE-2026-32631**, involves symlink type auto-detection triggering an SMB connection to a UNC path or drive-less absolute path (e.g., `\attacker\share`), exposing the user’s NTLMv2 hash. The fix modifies `compat/mingw.c` to skip auto-detection for symlink targets starting with a backslash, defaulting to file symlinks and warning users to set `symlink=dir` in gitattributes if needed. The patch is already released in Git for Windows v2.53.0(3) and is now **queued for integration** into upstream Git.

**Key participants**: Johannes Schindelin, Junio C Hamano.
**Status**: Queued for integration.

---

### `git repack --drop-filtered` for partial clones (RFC v2)
Siddharth Shrimali’s RFC v2 series introducing `--drop-filtered` to `git repack` for partial clones is now **ready for substantive review**. The feature allows users to safely reclaim disk space by removing locally cached promisor blobs exceeding a user-specified size threshold (e.g., `--filter=blob:limit=10M`). The v2 update fixes the `--write-bitmap-index` validation flaw, implements critical safety guards (merge/rebase/cherry-pick checks and index validation), and adds comprehensive documentation. The implementation adds two new options: `--drop-filtered` (real runs) and `--dry-run` (reporting only). It introduces `repack-filtered.c` for the core logic, extends `list-objects-filter` with `list_objects_filter__filter_oidset()`, and modifies `repack-promisor.c` to support excluding objects during promisor pack rebuilding. Safety is guaranteed by construction: only objects marked as promisor (recoverable from the remote) are targeted, and locally created objects are untouched.

**Key participants**: Siddharth Shrimali, Junio C Hamano.
**Status**: Ready for review; RFC v2 posted.

---

### Git 3.0: lowercase-only hex object IDs (RFC)
Brian m. carlson’s RFC series proposing a Git 3.0 breaking change to enforce lowercase-only hex object IDs continues to draw **maintainer scrutiny and philosophical debate**. Junio expanded his earlier Robustness Principle objection with a concrete example: Git’s dumb HTTP transport encountering a loose object directory named in uppercase (e.g., `objects/AB/`). He argues that a robust implementation should accept and normalize such inputs rather than reject them outright, suggesting a potential compromise that preserves case-insensitive parsing for external inputs while enforcing lowercase internally. Jeff King flagged an edge case: uppercase hex in object payloads (e.g., commits) creates parallel histories, as Git does not normalize case when hashing contents. Brian countered by citing modern security practices (TLS, HTTP request smuggling) to argue that lenient parsing is now a security anti-pattern. The discussion remains focused on **design philosophy** rather than implementation details, with no maintainer decision yet.

**Key participants**: Brian m. carlson, Junio C Hamano, Jeff King.
**Status**: Under debate; no decision yet.

---

### `git add --resolved` design debate
Junio C Hamano’s **4-patch series** introducing `git add --resolved`, which stages only paths whose conflict markers have been removed, sparked a **philosophical divide** over Git’s interface conventions. Michael Montalbo argued that the option name **conflates selection (unmerged paths) with policy (conflict-marker check)**, proposing an alternative design: a general `--unmerged` selector with explicit policy flags like `--allow-conflict-markers` or `--skip-conflict-markers`. Junio defended the workflow-oriented `--resolved` option, citing Git’s existing conventions (e.g., `git am --resolved`) and arguing that the monolithic approach reflects user intent. The debate highlights a tension between Git’s traditional workflow-oriented commands and a more modular approach that exposes underlying implementation details. No consensus has been reached, and the discussion may continue with further arguments or counter-proposals.

**Key participants**: Junio C Hamano, Michael Montalbo.
**Status**: Under debate; v3 posted.

---

## In brief

**`git rebase --update-refs` symref bugfix** -- Son Luong Ngoc’s series fixing `git rebase --update-refs` behavior with symrefs **resolved its design debate** after Phillip Wood provided concrete evidence (TopGit’s use of HEAD symrefs) that the edge case is actively relied upon. The series is now **ready for merging** after addressing minor refinements.

**`git cat-file --batch-command` assertion fix** -- Jeff King’s fix for a crash in `git cat-file --batch-command` when omitting `%(objecttype)` from the format string is **queued for `next`**, resolving a long-standing issue.

**Pack-bitmap off-by-one edge case** -- David Lin’s bugfix for `find_objects()` in the pack-bitmap subsystem, where objects at position zero were incorrectly excluded, is **accepted for integration**.

**UTF-8 string-width refactoring withdrawn** -- Hardik Kumar’s effort to change the return type of `utf8_strwidth()` and `utf8_strnwidth()` from `int` to `size_t` was **withdrawn** after sustained opposition. The thread concluded with a **documentation patch** explaining the rationale for the `int` return type.

**ODB performance regression discussion** -- Patrick Steinhardt proposed a **fundamental architectural shift** to move alternates handling into the "files" backend itself, treating alternates as an implementation detail rather than a cross-backend concern. The proposal has gained **explicit endorsements from Peff and Justin Tobler**.

**`git mv` symlink-handling fix** -- Lucas Zamboni Orioli’s series fixing misleading errors in `git mv` and rejecting tracked symlinks in the destination path is **ready for integration** after addressing Junio’s refactoring request.

**`git repo info` path keys** -- K Jayatheerth’s series adding path-related keys to `git repo info` is **under review**, with a forward-compatibility issue (use of `startup_info->prefix`) blocking progress.

**`git maintenance` regression fix** -- David Lin’s patch restoring the documented `"none"` strategy behavior in `git maintenance` is **queued for `next`**.

**`git add -e` refactoring** -- Gatla Vishwahwar Reddy’s refactoring of `git add -e` to replace a subprocess call to `git apply` with internal API is **approved for `next`**.

**`git index-pack` race-condition fix** -- Ted Nyman’s fix for concurrent download corruption in HTTP transport is **ready for merge** after receiving final approval from Peff.

**`git bisect --reset-when-found`** -- Harald Nordgren’s series adding an auto-reset option to `git bisect` is **ready for `next`** after addressing all surface-level feedback.

**`git config --global` documentation discrepancy** -- Nils Fahldieck reported a bug where `git config --global` reads only one file despite the documentation claiming it reads both `~/.gitconfig` and the XDG config file. The maintainer’s preference for documenting the single-file behavior remains unchallenged.

**Promisor-pack performance patch** -- Arijit Banerjee’s patch to speed up `git index-pack` for promisor packs by 15–26% became the focal point of a **policy debate** about AI-assisted contributions. The patch is stalled pending maintainer guidance on whether the performance benefit justifies an exception to the project’s policy.

**`diff-provider` RFC (v7)** -- Michael Montalbo’s RFC proposing a unified abstraction layer for diff hunks is **technically complete for its scope**, awaiting design feedback. The series ships two example providers: an in-process cache and an external process driver.

**`git history squash` v11** -- Harald Nordgren’s `git history squash` series is **functionally complete** and ready for integration, with message editing now the default behavior.

**Packfile URI race-condition fix (v6)** -- Ted Nyman’s fix for race conditions in HTTP transport is **fully approved** and ready for merge.

---

## Looking ahead

- **`git stash reword` reflog API redesign**: The `refs_reflog_edit_in_bulk()` API proposed by Junio is a significant new addition to Git’s internals. Track this for potential unification with other reflog operations (e.g., `stash drop`, `reflog expire`).
- **`git replay --linearize` per-branch support**: Toon Claes’s `--ref` proposal for per-branch linearization remains a potential future direction for the `git replay --linearize` feature.
- **ODB abstraction follow-ups**: Patrick Steinhardt’s ODB abstraction series continues to generate substantive review feedback. Track the `create_on_disk` callback design and initialization sequence discussions for future iterations.
- **Lowercase-hex RFC**: The philosophical debate over brian m. carlson’s lowercase-hex proposal may require **PLC input** to resolve.
- **`git repack --drop-filtered`**: Siddharth Shrimali’s RFC v2 series is ready for substantive review. The core functionality is complete, and the deferred work (drop log, remote verification) is explicitly scoped.