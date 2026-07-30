# The day in brief

**2026-07-29** was a **heavy-traffic day** with **72 emails across 24 threads**, dominated by **substantive reviews, bugfixes, and design debates**. The most urgent items:
- **`git branch --delete-merged` (Harald Nordgren’s v23)** is **blocked on two correctness issues** (stacked-branch protection and upstream push-check logic) identified by Phillip Wood. The series is otherwise ready for merging.
- **`git history squash` (Toon Claes)** was **ejected from `next`** after Phillip Wood exposed **fundamental flaws in reachability logic** and UX inconsistencies. Fixups are in progress.
- **Junio proposed a temporary moratorium on new topics in `seen`** unless they receive substantive review, a direct response to the `git history squash` episode.

Other notable threads:
- **`git rebase --update-refs` symref bugfix (Son Luong Ngoc)** saw **design resolution** after Phillip Wood provided concrete evidence (TopGit’s use of HEAD symrefs) that the edge case is actively relied upon.
- **`git cat-file --batch-command` assertion fix (Jeff King)** was **queued for `next`**, resolving a long-standing crash.
- **`pack.allowDuplicateObjects` RFC (friel@openai.com)** remains **contentious**, with Junio opposing the feature on principle and Taylor Blau defending it as a pragmatic optimization.

---

## Notable threads

### `git branch --delete-merged` v23: blocked on correctness issues
**Headline**: Harald Nordgren’s **7-patch series** adding `--delete-merged` to `git branch` is **ready for merging** but **blocked on two technical issues** identified by Phillip Wood:
1. **Stacked-branch protection test discrepancy**: The test expects the upstream config of a kept merged branch (`lower`) to remain intact, but the design goal is to **clear stale configs** for kept branches whose own upstream is being deleted.
2. **`branch_pushes_to_upstream()` correctness**: The current implementation risks **false negatives** (deleting branches that should be preserved, e.g., `main` tracking `origin/main`) or **false positives** (keeping branches that should be deleted).

**Why it matters**: The series provides **safe automated branch cleanup** with **order-independent stacked-branch protection**, per-branch opt-out, and `--dry-run` preview. The blocking issues undermine its **safety guarantees**, but the core design (flags-based API, `--forked` filter, repeatable `--delete-merged` arguments) is sound.

**Next steps**: Harald must **rewrite the push-detection logic** and **reconcile the stacked-branch test discrepancy** before the series can proceed. Junio has **confirmed the series is ready for merging** pending these fixes.

---

### `git history squash` ejected from `next` after fundamental flaws exposed
**Headline**: Toon Claes’s **`git history squash`** subcommand was **ejected from `next`** after Phillip Wood’s review exposed **critical correctness flaws** in its reachability logic:
- The implementation **incorrectly assumes `UNINTERESTING` commits imply a `BOTTOM` commit** or preclude root commits.
- It **fails to handle multi-tip histories** (e.g., `git history squash ^A C D` in `-A-B-C \ D`).
- The `--reedit-message` option is **misleading**, as it creates a new commit rather than re-editing an existing one.

**Why it matters**: The command is part of the new `git history` suite, which shares infrastructure with `git rebase` and `git replay`. The flaws could lead to **silent data loss** or **incorrect squashing**, and the UX inconsistencies (e.g., refusing to squash tagged commits) complicate its use.

**Next steps**: Phillip is **developing fixups** to address the reachability logic, but the series may require **significant rework**. Junio’s ejection signals **heightened scrutiny for future `next` topics**, and he has **proposed a temporary moratorium on new `seen` topics** unless they receive substantive review.

---

### `git rebase --update-refs` symref bugfix: design resolved
**Headline**: Son Luong Ngoc’s **2-patch series** fixing `git rebase --update-refs` behavior with symrefs **resolved its design debate** after Phillip Wood provided **concrete evidence** (TopGit’s use of HEAD symrefs pointing to `refs/top-bases/...`) that the edge case is **actively relied upon** by real-world tools.

**Why it matters**: The series ensures `git rebase --update-refs` **handles symrefs robustly**, including:
- HEAD aliases (e.g., `main` pointing to `master`).
- Non-branch symrefs (e.g., tags, notes).
- Cross-worktree conflicts.

**Next steps**: The series is **ready for merging** after addressing minor refinements (explanatory comments, test organization). Junio has **conceded the design argument**, and the patch’s approach (using `resolved_ref` for HEAD comparison) will be retained.

---

### `pack.allowDuplicateObjects` RFC: philosophical impasse
**Headline**: friel@openai.com’s **RFC patch** proposing `pack.allowDuplicateObjects` and `--[no-]allow-duplicate-objects` for `git index-pack` remains **contentious**, with Junio opposing the feature on principle and Taylor Blau defending it as a **pragmatic optimization**.

### Key arguments

- **Junio**: Duplicate objects violate Git’s **integrity guarantees** and should never be allowed, even as an opt-in exception. He suggests **deduplicating during `index-pack`** instead of storing duplicates.
- **Taylor Blau**: Git’s internals are **already hardened against duplicates** (via `tb/pack-with-duplicates`), and the feature is a **defensive measure** for servers streaming overlapping packs. He proposes a **narrower design** that only relaxes the duplicate-OID check in `--strict` mode.

**Why it matters**: The feature addresses a **performance bottleneck** for servers (e.g., concatenating packs to avoid deduplication overhead), but Junio’s opposition suggests it may **struggle to gain traction** unless the author can demonstrate that deduplication during indexing is **unfeasible**.

**Next steps**: The author has **agreed to explore deduplication during `index-pack`**, but the philosophical debate remains unresolved. The thread may pivot to **implementation details** (e.g., how to deduplicate efficiently) rather than the feature’s existence.

---

### `git add --resolved`: design critique
**Headline**: Junio’s **4-patch series** introducing `git add --resolved` received a **substantive design critique** from Michael Montalbo, who argues the option name **conflates selection (unmerged paths) with policy (conflict-marker check)**.

### Key points

- **Current design**: `--resolved` stages only unmerged paths whose conflict markers have been removed.
- **Proposed alternative**: Split into `--unmerged` (selects unmerged paths) with explicit policy flags like `--allow-conflict-markers` or `--skip-conflict-markers`.

**Why it matters**: The critique targets a **core UX decision**—whether the command should be **monolithic** (like `--resolved`) or **modular** (like `--unmerged` with policy flags). The current implementation is **technically sound** but may **limit future extensibility**.

**Next steps**: Junio has not yet responded to the critique. The series is **otherwise ready for merging** after addressing minor fixes (option incompatibility logic, test infrastructure).

---

## In brief

### Bugfixes and refactoring

- **`git cat-file --batch-command` assertion fix** (Jeff King) – **Queued for `next`**, resolving a long-standing crash when omitting `%(objecttype)`.
- **`git maintenance` regression fix** (David Lin) – Restores the documented `"none"` strategy behavior, broken since October 2025.
- **`git add -e` refactoring** (Gatla Vishwahwar Reddy) – Replaces subprocess call to `git apply` with internal API, **approved for `next`**.
- **`git index-pack` race-condition fix** (Ted Nyman) – **Ready for merge**, addressing concurrent download corruption in HTTP transport.
- **`git repo info` path keys** (K Jayatheerth) – **Under review**, adding `path.toplevel`, `path.superproject-working-tree`, and others. Blocked on **ODB abstraction concerns** and `/dev/null` handling.
- **`git branch -d` bisect error message** (René Scharfe) – **Cooking in `next`**, clarifying error messages for branches used by `git bisect`.

### Performance and platform fixes

- **macOS regex leak workaround** (Chungmin Lee) – **Ready for merge**, preventing gigabyte leaks in `git grep` over binary files.
- **sparse-index ITA crash fix** (Derrick Stolee) – **Cooking in `next`**, preventing segfaults when collapsing sparse indexes with intent-to-add entries.

### Documentation and tests

- **`git merge-base --is-ancestor` tests** (Nikolaus Schuetz) – **Cooking in `seen`**, adding comprehensive test coverage.
- **`test_commit_body()` helper** (Shlok Kulshreshtha) – **Queued for `next`**, replacing 61 duplicated `git cat-file | sed` patterns.
- **Documentation grammar fix** (Ville Skyttä) – **Under review**, replacing "a *" with "an *" in man pages.

---

## On the radar

- **Junio’s moratorium on new `seen` topics**: The proposal to **halt new topics in `seen`** unless they receive substantive review remains **open-ended**. If adopted, it would **slow the intake of new work** but could **improve review quality** and reduce the risk of flawed topics reaching `next`.
- **`git history` suite follow-ups**: Phillip Wood’s fixups for `git history squash` may **affect other commands** in the suite (e.g., `git history replay`), given their shared reachability logic.
- **ODB abstraction concerns**: K Jayatheerth’s `git repo info` series raises **forward-looking questions** about exposing object database paths in light of **pluggable ODB backends**. Patrick Steinhardt (ODB abstraction lead) has not yet weighed in.