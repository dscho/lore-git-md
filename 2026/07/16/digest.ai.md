Here is the daily digest for the Git mailing list on **2026/07/16 (UTC)**.

---

### The day in brief
A **heavy-traffic day** (126 emails, 38 active threads) with **two clear highlights**:
1. **`git replay --linearize`** finally resolves its multi-branch ambiguity debate, clearing the last blocker for merging.
2. **`git cat-file --batch-command` remote-object-info** graduates to `next` after 18 iterations, marking a major milestone for GSoC.

The day also saw **performance optimizations** (`git last-modified`, `git stash rename`), **refactoring** (`the_repository` removal, `writev(3p)` revival), and **new features** (`git repo info` path keys, `git bisect --auto-reset`). Controversy flared briefly over `git stash rename`’s design, but the thread pivoted toward a consensus path.

---

### Notable threads

#### `git replay --linearize` resolves multi-branch ambiguity
**Headline**: The `--linearize` option for `git replay` will now **restrict to a single positive ref**, addressing usability concerns raised by Elijah Newren and Junio Hamano. The change ensures predictable behavior when replaying a single branch with merges, avoiding the unintended concatenation of unrelated branches.

**What happened**: Toon Claes posted a v8 iteration that adopts Elijah’s **minimal fix**—limiting `--linearize` to one positive ref—while deferring per-branch linearization. The commit message now explicitly warns users about the restriction, and tests verify the new behavior. Junio’s earlier syntax proposals (e.g., limiting to one negative ref plus multiple positives) are set aside in favor of simplicity.

**Why it matters**: This resolves the last substantive objection to a series that has been cooking since June. The feature is now **technically complete** and likely to graduate to `next` in the next cycle.

---

#### `git cat-file --batch-command` remote-object-info graduates to `next`
**Headline**: After **18 iterations**, Pablo Sabater’s GSoC project to integrate `git cat-file --batch-command` with remote object queries is **queued for `next`**. The series enables clients to query object metadata (e.g., size) from remotes without downloading the full object, with robust security checks and a dynamic allow-list for future extensibility.

**What happened**: Junio’s "What’s cooking" report confirms the series is **cleared for `next`**, with all prior feedback addressed. The final design forces a "size" request (discarding the result) when only `%(objectname)` is queried, ensuring the server validates the OID’s existence. The dynamic allow-list for metadata fields (e.g., `objecttype`) is now finalized, and CI passes.

**Why it matters**: This is a **major GSoC milestone**, demonstrating the project’s ability to shepherd complex features through rigorous review. The feature will benefit partial clones and CI pipelines by reducing unnecessary object downloads.

---

#### `git stash rename` design pivots toward consensus
**Headline**: The proposed `git stash rename` subcommand faces **maintainer skepticism** over its name and workflow, but a path forward emerges: the feature will likely be renamed to `git stash reword` and focus on **leaving the working tree on the first bad commit** (the culprit) after bisect, aligning with Junio’s expectations.

**What happened**: Harald Nordgren’s v1 series introduced `--auto-reset` to return to the original commit after bisect, but Junio argued this was counterintuitive for debugging. Brian m. Carlson provided a concrete use case (frequent stashing with default messages), and the discussion now centers on **renaming the option** (e.g., `--keep-culprit`) and adjusting the behavior to match user intent.

**Why it matters**: The thread highlights Git’s **user-centric design process**, where even small features undergo rigorous debate. The outcome will shape how users interact with stashes in automated workflows.

---

#### Performance optimizations land
**Headline**: Two performance patches address real-world bottlenecks:
1. **`git last-modified`**: Jeff King (Peff) and Toon Claes collaborated on a **4× speedup** (64 ms → 16 ms) by reusing Bloom filter logic from `revision.c`. The patch is now ready for review.
2. **`wt-status` quadratic insertion**: Sahitya Chandra’s patch replaces O(n²) `string_list_insert()` calls with O(n log n) sorting, improving status enumeration in large repositories.

**Why it matters**: Both patches target **scalability pain points** (monorepos, CI pipelines) and demonstrate Git’s responsiveness to performance regressions.

---

#### `the_repository` removal continues
**Headline**: Three refactoring series advance the `the_repository` removal effort:
1. **`refs` subsystem**: Patrick Steinhardt’s 8-patch series (merged to `master`) removes `the_repository` from refs backends and worktree code.
2. **`refspec.c`**: A 3-patch series (under review) makes refspec parsing explicitly accept a hash algorithm parameter.
3. **`copy.c` utilities**: A standalone patch (v2) refactors `copy_file()` to accept a `struct repository *`.

**Why it matters**: These **mechanical but critical** changes reduce global state, improving modularity and paving the way for multi-repository support.

---

### In brief
- **`git repo info` path keys**: K Jayatheerth’s GSoC series adds 7 new keys (e.g., `path.toplevel.absolute`, `path.git-prefix`) to expose repository paths in a scriptable format. Junio requested minor fixes to the final patch.
- **`git bisect --auto-reset`**: Harald Nordgren’s RFC proposes an option to automatically reset after bisect, targeting CI pipelines. Junio’s feedback questions the workflow’s necessity.
- **`writev(3p)` revival**: Patrick Steinhardt’s 6-patch series reintroduces a `writev(3p)` wrapper for performance, with debate over API naming (`git_writev()` vs. `git_write_gather()`).
- **`git repack --drop-filtered`**: Siddharth Shrimali’s RFC for partial clones adds a drop log to track removed promisor blobs. Junio identified a **blocking flaw** in bitmap validation.
- **`git submodule` + sparse-checkout**: Brad Larson’s RFC asks whether `git submodule` should respect sparse-checkout cones. No consensus yet.
- **`git fast-import` libification**: Christian Couder’s 7-patch series refactors the usage string and reduces globals. Junio flagged an **awkward intermediate state** in the final patch.

---

### On the radar
- **`diff.<driver>.process`**: Michael Montalbo’s RFC for external diff tools is **unblocked** after resolving a merge conflict with `mm/line-log-limited-ops`. A v6 is expected soon.
- **`trust_executable_bit`/`has_symlinks` refactoring**: Tian Yuchen’s v6 series (part of `the_repository` removal) faces a **design-level objection** from Junio over `ce_mode_from_stat()`’s relocation. Resolution may require keeping the function inline in the header.
- **`fetch_if_missing` refactoring**: Tian Yuchen’s patch to move the flag into `struct repository` needs to address a **behavior change** in repository-less invocations (e.g., `git index-pack`).

---

### Editorial notes
- **Traffic texture**: The day was **heavily weighted toward refactoring and performance**, with fewer new features than usual. The `the_repository` removal effort dominated, accounting for **20% of active threads**.
- **Controversy**: The `git stash rename` and `git bisect --auto-reset` threads sparked **design debates**, but both pivoted toward consensus by day’s end.
- **GSoC progress**: Pablo Sabater’s `remote-object-info` series and K Jayatheerth’s `git repo info` path keys demonstrate **strong mentorship**, with both projects nearing completion.