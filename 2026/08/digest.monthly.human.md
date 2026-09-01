# The Git Project -- Monthly Digest for August 2026

## The period in brief

August 2026 was an **exceptionally active and consequential month** for Git development, with **over 1,500 emails across 200+ threads** spanning five weeks. Traffic was **heavy, technically dense, and urgent in tone**, dominated by **architectural debates, regression fixes, and long-running feature series** reaching critical milestones. The standout developments: **`git branch --delete-merged` landed in `next`** after a 25-iteration journey, **`git history squash` was ejected from `next`** due to fundamental correctness flaws, and **Junio C Hamano proposed a sweeping architectural redesign** of built-in command interfaces. **Security vulnerabilities** (Git for Windows credential exfiltration, `gitk` command injection) and **performance regressions** (`git maintenance` geometric repacking, `git stash show` use-after-free) demanded immediate attention. The month also saw **major progress on the ODB abstraction effort**, with two series graduating to `master`, and **emerging policy debates** about AI-generated contributions and review culture.

---

## Key developments

### `git branch --delete-merged` lands in `next` after 25 iterations
Harald Nordgren’s **25-patch series** adding `--delete-merged` to `git branch` **graduated to `next`** after six months of review. The feature provides **safe, automated local branch cleanup** with **order-independent stacked-branch protection**, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview. The v24 update fixed two critical issues: **corrected push-detection logic** in `branch_pushes_to_upstream()` and **simplified stacked-branch protection** that clears upstream configs for kept branches whose own upstream is being deleted. The implementation supports repeatable `--delete-merged` arguments (e.g., `--delete-merged origin/main --delete-merged origin/next`) and optional positional patterns (e.g., `topic-*`). **Why it matters**: This is the **first major branch-management feature since `git switch`/`restore`**, addressing a long-standing pain point while preserving critical safety checks.

**Key participants**: Harald Nordgren (author), Phillip Wood (reviewer), Junio C Hamano (maintainer).
**Status**: **Merged to `next`**; expected to graduate to `master` in the current cycle.

---

### `git history squash` ejected from `next` after fundamental flaws exposed
Toon Claes’s `git history squash` subcommand, part of the new `git history` suite, was **ejected from `next`** after Phillip Wood’s review exposed **critical correctness flaws** in its reachability logic. The implementation incorrectly assumed `UNINTERESTING` commits imply a `BOTTOM` commit or preclude root commits, and failed to handle multi-tip histories (e.g., `git history squash ^A C D` in `-A-B-C \ D`). The `--reedit-message` option was also misleading, as it created a new commit rather than re-editing an existing one. Junio’s ejection signals **heightened scrutiny for future `next` topics**, and he proposed a **temporary moratorium on new `seen` topics** unless they receive substantive review. The series may require **significant rework**, with a maintainer ruling on OID case-sensitivity in autosquash markers now resolving the last open design question.

**Key participants**: Toon Claes (author), Phillip Wood (reviewer), Junio C Hamano (maintainer).
**Status**: **Ejected from `next`**; fixups in progress. A v15 reroll is expected soon.

---

### Architectural redesign of built-in command interfaces
Junio C Hamano proposed a **sweeping architectural change** to built-in command interfaces, replacing the `struct repository *repo` parameter in all built-in commands with a simple `bool has_repo` flag. The proposal, accompanied by an 8-patch series refactoring `git checkout`, `git switch`, and `git restore`, argues that the current design creates a **false promise of libification**, risks segfaults (e.g., `cd / && git foo -h`), and obscures the architectural boundary between built-ins (one-time orchestrators) and reusable utility code. The change affects 135 files and redirects the `the_repository` removal effort toward utility code outside `builtin/`, where libification is both feasible and meaningful. **Why it matters**: This is a **fundamental shift** in Git’s architecture, with implications for long-term maintainability and the scope of libification efforts.

**Key participants**: Junio C Hamano (author), Jeff King (reviewer), Elijah Newren (reviewer).
**Status**: **Under review**; the 8-patch series demonstrates the approach and may serve as a template for other built-ins.

---

### ODB abstraction work graduates to `master`
Two series advancing the **pluggable ODB effort** landed in `master`:
1. **Patrick Steinhardt’s 6-patch series** making on-disk ODB structures pluggable (delegating directory creation to backends).
2. **Justin Tobler’s 7-patch series** extending ODB transactions to support packfile writes in `git receive-pack`.

The latter series reached v5, addressing all prior feedback including a **critical lockfile lifecycle flaw** in the "files" backend via explicit `struct odb_source *` tracking. The series introduces `odb_transaction_finalize()` and `odb_transaction_commit_and_finalize_or_die()` to enforce proper transaction lifecycle management, and adds `odb_transaction_write_pack()`, the generic interface completing the ODB abstraction for `git receive-pack`. **Why it matters**: These are **major milestones** for the ODB abstraction effort, enabling alternative storage systems (e.g., cloud, database-backed ODBs). The `receive-pack` series makes the **first network-facing command fully backend-agnostic**.

**Key participants**: Patrick Steinhardt (author), Justin Tobler (author), Junio C Hamano (maintainer), Toon Claes (reviewer).
**Status**: **Both series merged to `master`**.

---

### Security vulnerabilities: Git for Windows and `gitk`
Two **critical security vulnerabilities** were addressed:
1. **Git for Windows credential exfiltration (CVE-2026-32631)**: Johannes Schindelin posted a **security fix** preventing credential exfiltration when cloning a crafted repository with symlinks enabled. The vulnerability involved symlink type auto-detection triggering an SMB connection to a UNC path or drive-less absolute path (e.g., `\attacker\share`), exposing the user’s NTLMv2 hash. The fix modifies `compat/mingw.c` to skip auto-detection for symlink targets starting with a backslash, defaulting to file symlinks and warning users to set `symlink=dir` in gitattributes if needed. The patch is already released in Git for Windows v2.53.0(3) and is now **queued for integration** into upstream Git.

2. **`gitk` command injection**: Tim Wiederhake’s patch adding user-configurable commands to `gitk`’s context menus was found to **interpolate placeholders into shell commands without quoting**, enabling arbitrary command execution via malicious commit titles. Junio C Hamano provided a concrete exploit example (`title?'; echo no'`). The author is evaluating two fixes: reverting to direct command execution (requiring wrapper scripts) or implementing proper escaping. **Why it matters**: Both issues highlight **Git’s ongoing attention to security**, with the `gitk` flaw being particularly concerning given its widespread use in corporate environments.

**Key participants**: Johannes Schindelin (author), Tim Wiederhake (author), Junio C Hamano (maintainer), Johannes Sixt (gitk maintainer).
**Status**:
- **Git for Windows**: Queued for integration.
- **`gitk`**: Fix pending; the feature is blocked until resolved.

---

### Performance regressions: `git maintenance` and `git stash show`
Two **high-impact performance regressions** were reported:
1. **`git maintenance` geometric repacking**: Stefan Haller reported two **blocking bugs**:
   - A **concurrency race** in `prune_packed_objects()` that removes fanout directories while concurrent writers attempt to create temporary files.
   - A **misconfigured auto-condition** (`geometric-repack.auto=100`) that triggers 256× more aggressively than documented, causing excessive background repacks in small repositories.
   The underlying race condition remains unaddressed, risking **data corruption** and **performance degradation** in routine operations.

2. **`git stash show` use-after-free**: Nicolas Le Cam reported a use-after-free regression in `git stash show` when `--src-prefix` or `--dst-prefix` is used, introduced in Git 2.52.0. The issue causes garbage in the diff header (e.g., `diff --git Uf.txt Uf.txt`) instead of the expected `diff --git a/f.txt b/f.txt`, breaking tools like `lint-staged` that rely on parseable diff output. The root cause is `OPT_STRING_F` storing pointers into `argv` elements that are later freed while `struct diff_options` still holds dangling pointers. **Why it matters**: Both regressions affect **production environments**, with the `git stash show` issue impacting scriptability.

**Key participants**: Stefan Haller (reporter), Nicolas Le Cam (reporter), Patrick Steinhardt (author), Junio C Hamano (maintainer).
**Status**:
- **`git maintenance`**: Unaddressed; may require a revert if no progress is made.
- **`git stash show`**: Root cause identified; fix expected soon.

---

### `git repack --drop-filtered` for partial clones (v5)
Siddharth Shrimali’s RFC v5 series introducing `--drop-filtered` to `git repack` for partial clones is now **feature-complete** and ready for substantive review. The feature allows users to **safely reclaim disk space** by removing locally cached promisor blobs exceeding a user-specified size threshold (e.g., `--filter=blob:limit=10M`). The v5 update fixes the `--write-bitmap-index` validation flaw, implements critical safety guards (merge/rebase/cherry-pick checks and index validation), and adds comprehensive documentation. The implementation adds two new options: `--drop-filtered` (real runs) and `--dry-run` (reporting only). **Why it matters**: This is a **long-awaited feature** for partial clone users, enabling disk-space management without breaking lazy-fetching.

**Key participants**: Siddharth Shrimali (author), Junio C Hamano (maintainer).
**Status**: **Ready for review**; RFC v5 posted.

---

### Policy debates: AI-generated contributions and review culture
Two **procedural debates** emerged:
1. **AI-generated contributions**: A technically complete patch series to replace deprecated `utime()` with `utimensat()` for POSIX.1-2024 compatibility is **blocked by policy** over AI-generated content. brian m. carlson raised a process objection, citing Git’s `SubmittingPatches` policy and the Developer Certificate of Origin’s requirement for contributors to know the origin of their code. The series was generated by an AI tool (Gemini Flash) and manually verified, but the policy currently prohibits significant AI-generated content due to legal and ethical concerns. Junio C Hamano acknowledged the legal risks, noting that accepting AI-generated work could expose the project to future legal costs if the code is later found to infringe. The discussion now centers on **balancing legal caution against contributor trust**, with no resolution in sight.

2. **Review culture**: Harald Nordgren and Phillip Wood clashed over the **pace of review iterations** in the `git history squash` thread. Phillip criticized rapid rerolls for fragmenting discussion, while Harald argued the current "resonant frequency" (multi-day waits) is suboptimal. **Why it matters**: These debates reflect **tensions between Git’s volunteer-driven culture and contributors’ expectations**, with the AI discussion being particularly relevant as the project grapples with **how to handle AI-generated code**.

**Key participants**: brian m. carlson (reviewer), Junio C Hamano (maintainer), Harald Nordgren (author), Phillip Wood (reviewer).
**Status**:
- **AI-generated contributions**: Blocked; may require PLC intervention.
- **Review culture**: Unresolved; the `git history squash` series remains stalled.

---

### Diff provider RFC pivots to "cousin" interface
Michael Montalbo proposed a **major pivot** in the diff provider RFC, abandoning the "diff provider" abstraction introduced in v7 in favor of a simpler "cousin" interface to `xdiff-interface.h`. The new approach models the interface after the existing `diff.<driver>.process` plumbing, letting users plug in external diff tools via pkt-line while preserving xdiff as the final arbiter for content rendering. This pivot addresses Phillip Wood’s core feedback by:
- Dropping the "authority" question (external tools are now advisory, not authoritative).
- Replacing invented terminology with established Git terms.
- Ejecting the `diff-hunks` store to avoid cache-keying concerns.
The discussion now centers on whether the "cousin" interface is the right abstraction for integrating external diff tools without disrupting Git’s built-in features like word diff and `--color-moved`.

**Key participants**: Michael Montalbo (author), Phillip Wood (reviewer), Junio C Hamano (maintainer).
**Status**: **Under review**; a reroll based on v6 is planned.

---

### `git organize` introduced for source tree restructuring
Michael Montalbo introduced `git organize`, a new command designed to **restructure Git’s source tree** by moving files into subsystem-specific directories (e.g., `odb/`, `refs/`, `pack/`) based on recorded placement rules in a `.gitorganize` file. The 14-patch RFC series includes both the tool itself and a demonstration of its use to reorganize Git’s own codebase, carving out eight subsystems (`odb`, `refs`, `pack`, `diff`, `revision`, `index`, `setup`, `transport`) in separate commits. The tool externalizes file placement decisions into a tracked configuration file, enabling **auditable, reversible reorganizations**. It provides two main workflows:
- A **labeler** assigns semantic labels (e.g., `component=odb`) using project-specific heuristics.
- An **organizer** updates references (e.g., `#include` paths, build system entries) after moves.
The series demonstrates the tool’s practical application by mechanically moving each subsystem to its own directory and updating all references, leaving the tree buildable after each move. **Why it matters**: This is a **novel approach to modularity**, with potential to break the deadlock in Git’s long-running directory reorganization efforts.

**Key participants**: Michael Montalbo (author), Junio C Hamano (maintainer), Patrick Steinhardt (reviewer).
**Status**: **Under review**; the series may spark broader discussion about modularity and tool design.

---

## In brief

**`git stash reword`** -- Emin Özata’s `git stash reword` feature is **blocked by a fundamental correctness issue**: the current implementation slurps the entire reflog into memory, deletes it on-disk, and rewrites it, creating a critical window where the reflog exists only in memory. Junio proposed extending the reflog API to support **atomic replacement of individual entries** via `refs_reflog_edit_in_bulk()`, which would eliminate the risk and make the operation truly atomic. **Status**: Blocked; API redesign required.

**`git add --resolved`** -- Junio C Hamano’s **4-patch series** introducing `git add --resolved` (staging only paths whose conflict markers have been removed) sparked a **philosophical divide** over Git’s interface conventions. Michael Montalbo argued that the option name **conflates selection (unmerged paths) with policy (conflict-marker check)**, proposing an alternative design: a general `--unmerged` selector with explicit policy flags like `--allow-conflict-markers` or `--skip-conflict-markers`. **Status**: Under debate; v3 posted.

**`git cat-file --batch-command`** -- Pablo Sabater’s **GSoC series** extending `git cat-file --batch-command` to support `%(objecttype)` in remote-object-info queries **reached v6**, addressing all feedback. The series adds end-to-end support for object type metadata, the last piece guaranteed to match between client and server post-fetch. **Status**: Ready for integration.

**`git push --porcelain` inconsistency** -- Xavier Morel reported that `git push --delete --porcelain` outputs human-readable errors (e.g., "remote ref does not exist") instead of the documented machine-readable format. Junio proposed a fix using `!` as an error prefix. **Status**: Fix pending.

**Performance optimizations** -- Two performance-related series graduated:
1. **`paint_down_to_common()` optimization**: Kristofer Karlsson and Tian Yuchen’s **100–1000× speedup** for asymmetric merge-base queries (e.g., repositories with import grafts) was **queued in Junio’s tree**.
2. **`git last-modified` Bloom filter reuse**: Toon Claes’s **3.7× speedup** for `git last-modified` by reusing Bloom filters was **ready for review**.

**`git maintenance` promisor packs** -- Taylor Blau’s two-patch bugfix for geometric repacking was reviewed by Patrick Steinhardt, who noted a potential edge case (now resolved). **Status**: Ready for integration.

**`git-interpret-trailers` documentation** -- Kristoffer Haugsbakk’s 11-patch series overhauling the `git-interpret-trailers` man page is **fully merged to `master`**.

**`git repo info`** -- K Jayatheerth’s seven-patch v5 series adding path-related keys to `git repo info` faced architectural concerns from Junio C Hamano. The series duplicates logic between `git repo info` and `git rev-parse` for keys like `path.cdup`, `path.toplevel`, `path.superproject-root`, and `path.git-prefix`. **Status**: Under review; may require consolidation.

**`git commit --amend` during conflict resolution** -- Elijah Newren’s single-patch bugfix extending Git’s existing protection against `git commit --amend` during conflict resolution to cover `git am`, `git revert`, and all forms of `git rebase` received structural improvement suggestions from Phillip Wood. **Status**: Under review.

**`git worktree` CoW optimizations** -- Peter Morris’s RFC for filesystem-level copy-on-write in `git worktree add` was blocked by Brian M. Carlson’s report of **ReFS corruption risks**. **Status**: Blocked.

**`http.sslVerifyStatus`** -- Grayson Gordon’s v6 patch introducing OCSP validation (`http.sslVerifyStatus`) is **technically complete and ready for `next`**. The patch targets a niche but critical security gap: OpenSSL-linked Git binaries (common in FIPS-compliant deployments) currently ignore OCSP staples, leaving revoked certificates unchecked.

**`gitk` color-preference dialog** -- Johannes Sixt’s v2 series overhauling gitk’s color-preference dialog consolidated logic, refactored code, and improved usability. **Status**: Ready for integration.

**`git reflog` API** -- Patrick Steinhardt proposed extending reference transactions to handle reflog edits natively, countering Junio’s `refs_reflog_edit_in_bulk()` proposal. **Status**: Under discussion.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder’s series (replacing `GIT_NO_LAZY_FETCH=fromAccepted`) is stalled on submission format but addresses a key security objection. **Status**: Under review.

**`git diff -l`** -- Elijah Newren’s patch updating the short help message for `git diff -l` was marked for `next`. **Status**: Ready for integration.

**Bash completion slowdown** -- Matthew Hughes reported a **performance regression** in bash completion for large repositories, with `git ls-files` scans causing 1–2 s hangs. **Status**: Under investigation.

**`chdir_notify` API** -- Colin Hinton’s patch removing the unused `name` parameter from the `chdir_notify` API was updated to document historical context. **Status**: Ready for integration.

**`git whoami` command proposal** -- Andrew Pleeter’s proposal for a new `git whoami` command to display the current Git identity and signing configuration sparked debate about project scope. Both brian m. carlson and Junio C Hamano expressed skepticism, requesting machine-readable output, removal of `the_repository` usage, support for X.509 signing keys, and justification for why this doesn’t belong in `git var`. **Status**: Under debate.

**`git stash` branch-aware design** -- Vladimir Sitnikov’s RFC proposing a branch-aware or worktree-aware stash design to prevent silent interference between worktrees’ stash operations advanced with a reflog-based solution from Phillip Wood. **Status**: Under discussion.

**Unified `post-worktree` hook proposed** -- Domen Kožar proposed a unified `post-worktree` hook to address Junio C Hamano’s design objection to the `post-worktree-*` series. The compromise replaces three separate hooks with a single hook using a subcommand-style interface (`add`, `move`, `remove`) and passing all relevant paths and the worktree ID as arguments. **Status**: Under review.

---

## Looking ahead

The next month is likely to see continued progress on several fronts:

- **Architectural redesign of built-in command interfaces**: Junio’s proposal to replace `struct repository *` parameters with a `bool has_repo` flag will require careful review and coordination with the broader `the_repository` removal effort. The 8-patch series refactoring `git checkout`, `git switch`, and `git restore` may serve as a template for other built-ins.

- **`git history squash` series**: Harald Nordgren’s `git history squash` series is expected to see a v15 reroll incorporating the maintainer’s ruling on OID case-sensitivity. The series is functionally complete and likely to graduate to `next` once the reroll is posted.

- **Security vulnerabilities**: The `gitk` command injection flaw and `git stash show` use-after-free regression are likely to see fixes posted soon, given their impact on production environments.

- **ODB abstraction follow-ups**: The ODB abstraction effort will continue with follow-ups addressing transaction lifecycle and source-tracking design debt. The `objectStorage` extension (planned) will enable non-filesystem backends.

- **Policy debates**: The AI-generated content policy debate may require intervention from the Project Leadership Committee to clarify or amend `SubmittingPatches`. The review culture discussion may resurface in the context of other long-running series.

- **Performance regressions**: The `git maintenance` geometric repacking bugs remain unaddressed and may require a revert if no progress is made. The `git stash show` use-after-free regression is likely to see a fix soon.

- **`git repack --drop-filtered`**: Siddharth Shrimali’s RFC v5 series is ready for substantive review and may graduate to `next` in the coming weeks.

- **`git organize`**: Michael Montalbo’s `git organize` series may spark broader discussion about modularity and tool design, particularly in relation to Patrick Steinhardt’s `libgit.a` effort.

- **Diff provider RFC**: Michael Montalbo’s pivot to a "cousin" interface for external diff tools will need community feedback to determine whether the new abstraction meets the original goals without disrupting Git’s built-in features.