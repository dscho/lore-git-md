# Here is the daily digest for the Git mailing list on 2026/07/07 (UTC):

---

### The day in brief
A **heavy but productive** day on the Git mailing list, with **132 emails across 29 threads**. The standout developments: **Junio C Hamano approved and queued multiple series** (including a 12-patch Coverity leak-fix series and a 13-patch repository setup refactoring), **design debates advanced** on `git replay --linearize` and `git history squash`, and **performance regressions were diagnosed** in the reftable backend and commit-graph subsystem. The Rustification effort saw **build system adjustments for macOS** land, while **security inquiries** about Git for Windows and libssh2 were resolved. The day’s tone was **collaborative and forward-looking**, with maintainers and contributors aligning on next steps for long-running efforts.

---

### Notable threads

#### 1. **Coverity leak fixes land** (`js/coverity-leak-fixes`)
**What happened**: A 12-patch series from Johannes Schindelin addressing memory, file-descriptor, and process-handle leaks flagged by Coverity Scan was **approved and queued for merging** by Junio C Hamano. The series plugs leaks in core subsystems (loose-object loading, bundle-uri downloads, run-command, diff-lib, line-log, dir, submodule, worktree, imap-send, reftable, and Windows process termination) with no user-visible behavior changes.
**Why it matters**: This is a **critical hygiene effort** to harden Git’s codebase against resource leaks, particularly in error paths. The series includes **680 lines of new tests** and addresses real-world issues (e.g., a leak in `read_one_dir()` under `GIT_TEST_COMMIT_GRAPH_CHANGED_PATHS=1`).
**Current status**: **Merged to `next`**. All patches carry Reviewed-by from Patrick Steinhardt, Jeff King, Eric Sunshine, and Junio.
**Key quote**: *"The v2 iteration looks good. Thanks, both."* — Junio C Hamano.

---

#### 2. **Repository setup refactoring advances** (`ps/setup-split-discovery-and-setup`)
**What happened**: Patrick Steinhardt’s 13-patch series to **decouple repository discovery from setup** received **maintainer approval** after Justin Tobler’s sign-off. The series introduces `struct repo_discovery` to hold discovery results, enabling distinct phases and reducing scattered setup logic. Junio confirmed he will "replace" the series in his integration branches.
**Why it matters**: This is a **foundational refactoring** for the ODB abstraction effort and `the_repository` removal, eliminating global state and improving clarity. The series touches `setup.c`, `repository.h`, and related headers, with no behavior changes.
**Current status**: **Queued for `next`**. Junio’s "Will replace" signals intent to merge pending no further issues.
**Key quote**: *"The changes in this version look good to me."* — Justin Tobler.

---

#### 3. **`git replay --linearize` design debate** (`tc/replay-linearize`)
**What happened**: Toon Claes’s v7 series adding `--linearize` to `git replay` (to flatten merge commits) **resolved its last major hurdle**—removing mutual exclusivity between `--linearize` and `--revert`—but Junio C Hamano raised a **new edge case**: combining the two options on a partial merge range (e.g., a range including a merge commit `M` and one side of the merge `A`, but excluding the other side `B` and the merge base `O`). The current implementation may not handle this case reliably.
**Why it matters**: The series is **technically complete** and addresses all prior feedback, but this edge case could **undermine the feature’s predictability**. The debate now centers on whether to **reinstate the restriction** or **demonstrate the edge case is handled**.
**Current status**: **Under review**. Junio’s tone suggests he is open to merging if the edge case is addressed, but the ball is in Toon’s court.
**Key quote**: *"I am not sure if the current code can reliably tell the difference between the two cases, and if it can, if it is easy to explain."* — Junio C Hamano.

---

#### 4. **`git history squash` usability questions** (`harald/history-squash`)
**What happened**: Harald Nordgren’s v7 series implementing `git history squash` (to collapse commit ranges into a single commit) **addressed all prior feedback** but faces **new usability concerns** from Phillip Wood. The unresolved questions:
   - Should `--reedit-message` (or `--edit`) be the **default** to encourage commit hygiene?
   - Should the `--reedit-message` template **reorder commits** to place `fixup!`/`squash!` markers after their targets (matching `git rebase -i`)?
   - Should the command **sanitize `rev-list` options** passed after `--` (e.g., `--reverse`) to prevent incorrect range resolution?
**Why it matters**: The series is **functionally complete** and marked "Will replace" by Junio, but these **UX gaps** could lead to confusion or incorrect results. The debate reflects broader tensions between **flexibility** and **safety** in Git’s history-rewriting tools.
**Current status**: **Awaiting v8**. Phillip’s feedback is **substantive and actionable**, but Harald has not yet committed to all requested changes.
**Key quote**: *"I think it would be better to reorder the commits so that the `fixup!`/`squash!`/`amend!` commits appear immediately after their target."* — Phillip Wood.

---

#### 5. **Reftable tombstone performance fix** (`kk/reftable-tombstone-quadratic-fix`)
**What happened**: Kristofer Karlsson’s 2-patch series fixing a **quadratic performance regression** in the reftable backend (when re-creating deleted refs) **resolved its downstream compatibility risk** with libgit2. The series moves tombstone handling from the merged iterator to call sites, enabling early termination when tombstones exceed the relevant bound. Performance improves from **~14s to ~0.2s** in the perf test’s 8000-ref scenarios.
**Why it matters**: This is a **critical fix** for the reftable backend, which is increasingly used in large-scale deployments. The series preserves libgit2 compatibility while maintaining the performance gain.
**Current status**: **Needs review**. Patrick Steinhardt identified **test design issues** (measurement isolation, tombstone carryover), which Kristofer agreed to address.
**Key quote**: *"The fix is narrowly scoped to the reftable backend."* — Thread summary.

---

#### 6. **Rustification build system adjustments for macOS** (`sn/osxkeychain-rust-universal`)
**What happened**: Shardul Natu and Koji Nakamaru’s v7 series adjusting the build system for Rust components on macOS **landed in Junio’s integration branches**. The series resolves a **parallel build race condition** in `git-credential-osxkeychain`, adds **Universal Binary support** (Intel + Apple Silicon), and enables **CI verification** for the macOS credential helper.
**Why it matters**: This is a **key enabler** for the Rustification effort, ensuring macOS users can build and test Rust components reliably. The series is **build-time only**, with no on-disk format changes.
**Current status**: **Queued for `next`**. Junio noted the v7 series is **functionally identical to v6**, which is already in flight.
**Key quote**: *"The series is now complete and ready for integration."* — Thread summary.

---

### In brief
- **`git rebase -i -x` debate**: Trevor Gross’s proposal to add `-x` to `git rebase -i` (to append "cherry picked from" trailers) **faced strong opposition** from Junio, Jeff King, and Phillip Wood, who argued the feature misaligns with `rebase`’s core semantics. The discussion **pivoted to extending `git cherry-pick` with an interactive mode** (`-i`), which Peff confirmed is feasible.
- **Config lock timeout**: Johannes Schindelin **nudged the stalled `core.configLockTimeout` patch**, clarifying his earlier naming feedback was supportive, not an objection. The patch remains **blocked on a technical tradeoff** between process-wide and per-repository semantics.
- **Commit-graph regression fix**: Kristofer Karlsson’s 2-patch series fixing a **performance regression in split commit-graph chains** (introduced in September 2025) was **approved for merging** after Junio’s clarification request. The fix restores performance from **4133ms to 233ms** for incremental writes.
- **`git_hash_*()` API hardening**: Jeff King’s 7-patch series to **make `git_hash_discard()` idempotent** and standardize hash API usage **received maintainer approval** after addressing cosmetic nits. The series adds defensive runtime checks and eliminates direct calls to low-level hooks.
- **`includeIf.worktree` symlink handling**: Patrick Steinhardt **endorsed Chen Linxuan’s proposal** to store a non-realpath worktree path in `struct repository`, resolving a behavioral inconsistency with `gitdir`. The v7 series is **ready for final review**.
- **ODB optimizations pluggable**: Patrick Steinhardt’s 11-patch series to **make ODB optimizations pluggable** received **substantive review** from Junio, who endorsed the design goals (hook expansion, task reordering) while raising minor test concerns. The series is **under active review**.
- **`unpack-trees` quadratic fix**: Henrique Ferreiro’s patch to **fix a quadratic-time index scan** in `next_cache_entry()` (triggered by `git diff` with early-index pathspecs) **faced a critical correctness concern** from Junio: the new `cache_bottom` optimization may break when index/tree ordering differs. The fix is **blocked pending resolution**.

---

### On the radar
- **`git replay --linearize` edge case**: Toon Claes must **demonstrate the edge case** (partial merge range with `--linearize` and `--revert`) is handled or **reinstate the restriction**.
- **`git history squash` UX gaps**: Harald Nordgren needs to **address Phillip Wood’s feedback** on default behavior, template reordering, and `rev-list` option sanitization in v8.
- **Reftable tombstone test design**: Kristofer Karlsson will **revise the perf test** to ensure measurement isolation and prevent tombstone carryover.
- **`unpack-trees` quadratic fix**: Henrique Ferreiro must **address Junio’s correctness concern** about the interaction between forward/backward scan logic.
- **HTTP fetch regression**: SZEDER Gábor’s report of an **unexpected full download** during incremental fetch remains **unresolved**, with Junio suggesting GitHub’s server-side behavior as a potential factor.

---

### Threads omitted
- **Translation updates** (French, Chinese) and **documentation fixes** (Rust support timeline, `t9811` test modernization) were covered in "In brief."
- **Security inquiries** (Git for Windows and libssh2 CVE-2026-55200) were resolved without action.
- **Trace2 ancestry test fix** and **reflog test modernization** were uncontroversial and ready for merging.