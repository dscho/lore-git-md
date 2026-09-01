# The Git Project -- Monthly Digest for August 2026

## The period in brief

August 2026 was an **exceptionally active and consequential month** for Git development, with **over 1,500 emails across 200+ threads** spanning five weeks. Traffic was **heavy, technically dense, and urgent in tone**, dominated by **regressions, security flaws, architectural debates, and long-running feature series reaching critical milestones**. The standout developments: **`git branch --delete-merged` landed in `next`** after a 25-iteration journey, **`git history squash` was ejected from `next`** due to fundamental correctness flaws, **two ODB abstraction series graduated to `master`**, and **a sweeping architectural redesign of built-in command interfaces was proposed**. Security concerns took center stage with **critical flaws in `gitk` and `git maintenance`**, while **policy debates** over AI-generated contributions and review culture highlighted tensions between Git’s volunteer-driven culture and modern development practices. **Do not miss**: the `git history squash` autosquash marker resolution policy, Junio C Hamano’s architectural redesign proposal, and the use-after-free regression in `git stash show`.

---

## Key developments

### `git branch --delete-merged` lands in `next` after 25 iterations
Harald Nordgren’s **25-patch series** adding `--delete-merged` to `git branch` **graduated to `next`** after six months of review, marking the **first major branch-management feature since `git switch`/`restore`**. The feature provides **safe, automated local branch cleanup** with **order-independent stacked-branch protection**, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview. The v24 update fixed two critical issues: **corrected push-detection logic** in `branch_pushes_to_upstream()` and **simplified stacked-branch protection** that clears upstream configs for kept branches whose own upstream is being deleted. The implementation supports **repeatable `--delete-merged` arguments** (e.g., `--delete-merged origin/main --delete-merged origin/next`) and **optional positional patterns** (e.g., `topic-*`). **Why it matters**: This addresses a long-standing pain point (local branch clutter) while preserving critical safety checks (checked-out branches, missing upstreams, push divergence). The stacked-branch protection design (abort-and-clear via single-pass logic) is a notable innovation that could influence future ref-handling commands.

---

### `git history squash` ejected from `next` after fundamental flaws exposed
Toon Claes’s `git history squash` subcommand, part of the new `git history` suite, was **ejected from `next`** after Phillip Wood’s review exposed **critical correctness flaws** in its reachability logic. The implementation incorrectly assumed `UNINTERESTING` commits imply a `BOTTOM` commit or preclude root commits, and failed to handle multi-tip histories (e.g., `git history squash ^A C D` in `-A-B-C \ D`). The `--reedit-message` option was also misleading, as it created a new commit rather than re-editing an existing one. Junio’s ejection signals **heightened scrutiny for future `next` topics**, and he proposed a **temporary moratorium on new `seen` topics** unless they receive substantive review. The series may require **significant rework**, with Harald Nordgren’s v14 addressing the final security blocker (case-insensitive OID matching) via a maintainer ruling requiring strict case-sensitive matching. **Why it matters**: The ejection underscores the **risk of premature integration** and the **importance of rigorous review** for history-rewriting commands. The ongoing debate about editor behavior (`--edit`/`--no-edit` vs. dynamic defaults) highlights tensions between **scriptability, UX consistency, and user intent**.

---

### ODB abstraction work graduates to `master`
Two series advancing the **pluggable ODB effort** landed in `master`:
1. **Patrick Steinhardt’s 6-patch series** making on-disk ODB structures pluggable (delegating directory creation to backends).
2. **Justin Tobler’s 9-patch v5 series** extending ODB transactions to `git receive-pack`, introducing `odb_transaction_write_pack()` and fixing a **critical lockfile lifecycle flaw** in the "files" backend.

**Why it matters**: These are **major milestones for the ODB abstraction effort**, enabling alternative storage systems (e.g., cloud, database-backed ODBs). The `receive-pack` series is particularly significant as it makes the **first network-facing command fully backend-agnostic**. The lockfile flaw could have left `.keep` files stranded, undermining transaction robustness.

---

### Architectural redesign of built-in command interfaces proposed
Junio C Hamano proposed a **sweeping architectural change** to built-in command interfaces, replacing the `struct repository *repo` parameter in all built-in commands with a simple `bool has_repo` flag. The proposal, accompanied by an 8-patch series refactoring `git checkout`, `git switch`, and `git restore`, argues that the current design:
- Creates a **false promise of libification** (built-ins are one-time orchestrators, not reusable libraries).
- Risks **segfaults** (e.g., `cd / && git foo -h`).
- Obscures the **architectural boundary** between built-ins and reusable utility code.

The change affects **135 files** and redirects the `the_repository` removal effort toward utility code outside `builtin/`, where libification is both feasible and meaningful. **Why it matters**: This is a **fundamental shift** in Git’s architecture, with implications for **safety, clarity, and long-term maintainability**. The `bool has_repo` design is motivated by safety and clarity, not efficiency, and is intended as a **final state for built-in command interfaces**.

---

### Security flaws in `gitk` and `git maintenance` addressed
Two **critical security issues** surfaced:
1. **`gitk` command injection**: Tim Wiederhake’s patch adding user-configurable commands to `gitk`’s context menus was found to **interpolate placeholders into shell commands without quoting**, enabling arbitrary command execution via malicious commit titles. Junio provided a concrete exploit example (`title?'; echo no'`). The author is evaluating fixes (escaping vs. direct execution).
2. **`git maintenance` geometric repacking bugs**: Stefan Haller reported two **blocking bugs** in `git maintenance`’s geometric repacking:
   - A **concurrency race** in `prune_packed_objects()` that removes fanout directories while concurrent writers attempt to create temporary files.
   - A **misconfigured auto-condition** (`geometric-repack.auto=100`) that triggers 256× more aggressively than documented, causing excessive background repacks in small repositories.

**Why it matters**: These bugs risk **data corruption** and **performance degradation** in routine operations. The geometric repacking feature was introduced in Git 2.54, making this a **regression with real-world impact**. The `gitk` issue is a **blocker** for the feature, highlighting Git’s **tension between usability and security** in UI tooling.

---

### `git repack --drop-filtered` for partial clones (v5)
Siddharth Shrimali’s **v5 series** introducing `--drop-filtered` to `git repack` for partial clones is **feature-complete** and ready for review. The feature allows users to **safely reclaim disk space** by removing locally cached promisor blobs exceeding a size threshold (e.g., `--filter=blob:limit=10M`). The v5 update fixes the `--write-bitmap-index` validation flaw, implements **critical safety guards** (merge/rebase/cherry-pick checks and index validation), and adds comprehensive documentation. The implementation adds two new options: `--drop-filtered` (real runs) and `--dry-run` (reporting only). **Why it matters**: This is a **long-awaited feature** for partial clone users, enabling disk-space management without breaking lazy-fetching. The series is **production-ready** but may see minor tweaks.

---

### Diff provider RFC pivots to "cousin" interface
Michael Montalbo proposed a **major pivot** in the diff provider RFC, abandoning the "diff provider" abstraction in favor of a simpler "cousin" interface to `xdiff-interface.h`. The new approach models the interface after the existing `diff.<driver>.process` plumbing, letting users plug in external diff tools via pkt-line while preserving xdiff as the final arbiter for content rendering. This pivot addresses Phillip Wood’s core feedback by:
- Dropping the "authority" question (external tools are now advisory, not authoritative).
- Replacing invented terminology with established Git terms.
- Ejecting the `diff-hunks` store to avoid cache-keying concerns.

**Why it matters**: The "cousin" interface may be the right abstraction for integrating external diff tools without disrupting Git’s built-in features like word diff and `--color-moved`. The discussion now centers on whether this meets the original goals.

---

### `git organize` introduced for source tree restructuring
Michael Montalbo introduced `git organize`, a new command designed to **restructure Git’s source tree** by moving files into subsystem-specific directories (e.g., `odb/`, `refs/`, `pack/`) based on recorded placement rules in a `.gitorganize` file. The **14-patch RFC series** includes both the tool itself and a demonstration of its use to reorganize Git’s own codebase, carving out eight subsystems (`odb`, `refs`, `pack`, `diff`, `revision`, `index`, `setup`, `transport`) in separate commits. The tool externalizes file placement decisions into a tracked configuration file, enabling **auditable, reversible reorganizations**. It provides two main workflows:
- A **labeler** assigns semantic labels (e.g., `component=odb`) using project-specific heuristics.
- An **organizer** updates references (e.g., `#include` paths, build system entries) after moves.

**Why it matters**: The tool’s **mechanical, reproducible methodology** avoids the arbitrariness of Patrick Steinhardt’s `lib/` approach while enabling incremental progress. The series raises questions about alignment with the `libgit.a` series (e.g., nesting subsystems under `lib/`).

---

### AI-generated content policy debate blocks POSIX.1-2024 compatibility
A **technically complete patch series** to replace deprecated `utime()` with `utimensat()` for POSIX.1-2024 compatibility is **blocked by a policy debate** over AI-generated contributions. brian m. carlson raised a process objection, citing Git’s `SubmittingPatches` policy and the Developer Certificate of Origin’s requirement for contributors to know the origin of their code. The series was generated by an AI tool (Gemini Flash) and manually verified, but the policy currently prohibits significant AI-generated content due to **legal and ethical concerns**. Junio acknowledged the legal risks, noting that accepting AI-generated work could expose the project to future legal costs if the code is later found to infringe. The discussion now centers on balancing **legal caution against contributor trust**, with no resolution in sight. **Why it matters**: This debate highlights the **tension between modern development practices and Git’s traditional policies**, with implications for future contributions.

---

### Use-after-free regression in `git stash show`
Nicolas Le Cam reported a **use-after-free regression** in `git stash show` when `--src-prefix` or `--dst-prefix` is used, introduced in Git 2.52.0. The issue causes **garbage in the diff header** (e.g., `diff --git Uf.txt Uf.txt`) instead of the expected `diff --git a/f.txt b/f.txt`, breaking tools like `lint-staged` that rely on parseable diff output. The root cause is `OPT_STRING_F` storing pointers into `argv` elements that are later freed while `struct diff_options` still holds dangling pointers. **Why it matters**: This affects **scriptability and production environments**, with the regression present since Git 2.52.0. The corruption is deterministic across versions but varies between runs in 2.53.0, suggesting heap memory reuse after free.

---

## In brief

**`git replay --linearize`** -- Toon Claes’s series resolved its long-standing design impasse over multi-branch ambiguity by restricting `--linearize` to a single positive ref, aligning with Elijah Newren’s "minimal fix" proposal. The series is **technically complete** and ready for `next`.

**`git stash reword`** -- Emin Özata’s feature is **blocked by a fundamental correctness issue**: the current implementation slurps the entire reflog into memory, deletes it on-disk, and rewrites it, creating a critical window where the reflog exists only in memory. Junio proposed extending the reflog API to support **atomic replacement of individual entries** via `refs_reflog_edit_in_bulk()`.

**Git for Windows security fix (CVE-2026-32631)** -- Johannes Schindelin’s patch preventing credential exfiltration when cloning crafted repositories with symlinks enabled is **queued for integration**. The vulnerability involves symlink type auto-detection triggering an SMB connection to a UNC path or drive-less absolute path (e.g., `\attacker\share`), exposing the user’s NTLMv2 hash.

**`git add --resolved`** -- Junio’s series adding `--resolved` to `git add` (staging only paths whose conflict markers have been removed) sparked a **philosophical divide** over Git’s interface conventions. Michael Montalbo argued the option name **conflates selection with policy**, proposing a modular `--unmerged` selector with explicit policy flags. No consensus has been reached.

**`git cat-file --batch-command`** -- Pablo Sabater’s **GSoC series** extending `git cat-file --batch-command` to support `%(objecttype)` in remote-object-info queries is **ready for integration**. The series adds end-to-end support for object type metadata, the last piece guaranteed to match between client and server post-fetch.

**`git maintenance` promisor packs** -- Taylor Blau’s two-patch bugfix for geometric repacking was reviewed by Patrick Steinhardt, who noted a potential edge case (now resolved).

**`git history` Bash completion** -- Vincent Mailhol’s v2 series added completion for the new `git history` command, with D. Ben Knoble advocating for a simplified approach.

**`git fetch_if_missing`** -- Tian Yuchen’s v6 series moving the global `fetch_if_missing` variable into `struct repository` is **mechanically complete** and ready for queuing.

**`git repo info`** -- K Jayatheerth’s v4 series adding path-related keys to `git repo info` faced a **design objection** to the `path.git-prefix` key, which exposes current-directory information rather than repository information.

**`git squash` proposal** -- A new `git squash <n>` command was proposed to combine the last `<n>` commits, sparking debate about redundancy with `git rebase --autosquash`.

**`git worktree` CoW optimizations** -- Peter Morris’s RFC for filesystem-level copy-on-write in `git worktree add` was blocked by Brian M. Carlson’s report of **ReFS corruption risks**.

**`git reflog` API** -- Patrick Steinhardt proposed extending reference transactions to handle reflog edits natively, countering Junio’s `refs_reflog_edit_in_bulk()` proposal.

**`git maintenance` geometric repacking race condition** -- Elijah Newren’s two-patch bugfix series addressing a race condition in Git’s geometric repacking mechanism was integrated into Junio’s `next` branch. The series fixes crashes in `git replay` and missing-object errors in other operations.

**`git organize`** -- Michael Montalbo’s new command for auditable source tree restructuring may spark broader discussion about modularity and tool design, particularly in relation to Patrick Steinhardt’s `libgit.a` effort.

**`git whoami` command proposal** -- Andrew Pleeter’s proposal for a new `git whoami` command to display the current Git identity and signing configuration sparked debate about project scope. Both brian m. carlson and Junio C Hamano expressed skepticism, requesting machine-readable output, removal of `the_repository` usage, support for X.509 signing keys, and justification for why this doesn’t belong in `git var`.

**`git stash` branch-aware design** -- Vladimir Sitnikov’s RFC proposing a branch-aware or worktree-aware stash design advanced with a reflog-based solution from Phillip Wood, enabling per-worktree isolation for detached HEAD worktrees without new configuration or ref namespaces.

**`git commit --amend` during conflict resolution** -- Elijah Newren’s single-patch bugfix extending Git’s existing protection against `git commit --amend` during conflict resolution to cover `git am`, `git revert`, and all forms of `git rebase` received structural improvement suggestions from Phillip Wood.

**`git worktree add` basename handling** -- René Scharfe’s four-patch v1 series fixing and cleaning up the `worktree_basename()` helper in `git worktree add` was approved by Junio C Hamano pending a commit-message tweak.

**`http.sslVerifyStatus` for OCSP stapling** -- Grayson Gordon’s v6 patch adding a boolean `http.sslVerifyStatus` option (default `false`) to enable OCSP staple validation via libcurl’s `CURLOPT_SSL_VERIFYSTATUS` is technically complete and ready for `next`.

**`gitk` AI contribution policy** -- Johannes Sixt’s documentation patch updating `gitk/README.md` to explicitly discourage AI-generated contributions aligned with upstream Git’s AI policy.

**`git whatchanged` deprecation feedback** -- The deprecation of `git whatchanged` (now requiring `--i-still-use-this`) does not disrupt workflows, as alternatives like `git log --oneline --name-status` suffice.

**Outreachy December 2026 cohort** -- Christian Couder invited volunteers, project ideas, and feedback on Git’s participation in Outreachy’s December 2026 cohort.

**Deprecation warning rewording** -- Junio’s v2 documentation patch reworded the user-facing deprecation warning in `usage.c` to eliminate misleading language that could imply the Git project might reverse a deprecation decision.

**`dk/use-nsec-runtime` series** -- D. Ben Knoble’s series converting `USE_NSEC` to a runtime option (`core.useNanosec`) is ready for `master`, but Junio raised a long-term architectural concern: the new dependency on `repo_config_values()` in `is_racy_stat()` could constrain future submodule support.

**Unified `post-worktree` hook proposed** -- Domen Kožar proposed a unified `post-worktree` hook to address Junio’s design objection to the `post-worktree-*` series, replacing three separate hooks with a single hook using a subcommand-style interface (`add`, `move`, `remove`).

---

## Looking ahead

The next month is likely to see continued discussion and potential resolution of several key topics:

- **Architectural redesign of built-in command interfaces**: Junio’s proposal to replace `struct repository *` parameters with a `bool has_repo` flag in built-in commands will require **careful review and coordination** with the broader `the_repository` removal effort. The 8-patch series refactoring `git checkout`, `git switch`, and `git restore` demonstrates the approach and may serve as a template for other built-ins.

- **`git history squash` series**: Harald Nordgren’s series is expected to see a v15 reroll incorporating the maintainer’s ruling on OID case-sensitivity. The series is **functionally complete** and likely to graduate to `next` once the reroll is posted.

- **AI-generated content policy**: The policy debate remains unresolved and may require **intervention from the Project Leadership Committee** to clarify or amend `SubmittingPatches`. The discussion may expand to address **contributor trust and disclosure practices**.

- **Use-after-free regression in `git stash show`**: The reported regression is likely to see a fix posted soon, given its impact on **scriptability and production environments**. The root cause (dangling pointers in `struct diff_options`) suggests a straightforward fix, but the regression’s presence since Git 2.52.0 may require backporting.

- **Diff provider RFC**: The pivot to a "cousin" interface will need **community feedback** to determine whether the new abstraction meets the original goals without disrupting Git’s built-in features.

- **`git organize`**: The series may spark broader discussion about **modularity and tool design**, particularly in relation to Patrick Steinhardt’s `libgit.a` effort.

- **Negative pathspec handling**: The design debate over whether negative pathspecs should be treated as absolute or relative to the full tree in `git ls-files` and `git add` remains unresolved. Junio’s alternative approach—calculating the common prefix across both positive and negative pathspecs—may require a new implementation or further discussion.

- **Trace2 hardening series**: Derrick Stolee’s series may see a v3 iteration addressing Jeff King’s architectural concerns, potentially dropping patch 3/4 entirely and focusing on the core fix.

- **`receive-report` hook**: Karthik Nayak’s hook for `git-receive-pack` is **queued in `seen`** and may graduate to `next` pending final review.

- **Geometric repacking race condition**: The integrated fixes may see follow-up work to address edge cases or performance optimizations.

The month ahead promises to be **technically rich and consequential**, with architectural shifts, policy clarifications, and long-awaited features poised to shape Git’s evolution.