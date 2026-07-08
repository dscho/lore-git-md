Here is the daily digest for the Git mailing list on 2026/07/07 (UTC):

---

## The day in brief

A **heavy-traffic Tuesday** (132 emails, 29 active threads) saw **two major series land**, **a critical regression fix**, and **lively design debates** on Rust integration, ODB abstraction, and rebase vs. cherry-pick workflows. The standout developments:

1. **Junio Hamano’s "What’s cooking" report** (2026/07/07/17-18-56) confirmed **14 topics graduated to `master`**, including Patrick Steinhardt’s promisor-remote auto-configuration and Elijah Newren’s ort merge backend hardening. The report also highlighted **33 "cooking" topics**, with Patrick’s repository setup refactoring (`ps/setup-split-discovery-and-setup`) and Taylor Blau’s geometric repack with cruft support (`tb/repack-geometric-cruft`) among the most consequential.
2. **A performance regression in the commit-graph subsystem** (2026/07/07/09-59-41) was **fixed and approved for merge**. The bug, introduced in September 2025, caused incremental commit-graph writes to slow from **233ms to 4133ms** on large repositories. Kristofer Karlsson’s two-patch series (now marked "Expecting a reroll" for a commit-message correction) restores the original behavior by ensuring the `topo_levels` slab is propagated to all layers of a split commit-graph chain.
3. **Jeff King’s 7-patch series hardening the `git_hash_*()` API** (2026/07/07/04-55-56) received **final approval** after addressing cosmetic nits. The series makes `git_hash_discard()` idempotent and adds defensive runtime checks to catch misuse early, aligning with industry-standard practices for hash libraries.
4. **Design debates raged** in three threads:
   - **Rust integration**: A proposal to make nanosecond timestamp support runtime-configurable (`core.useNsec`) gained traction, with Jeff King and Patrick Steinhardt endorsing the approach over the existing build-time knob (`USE_NSEC`). The discussion (2026/07/07/04-38-50) also touched on deprecating the build-time option entirely, though D. Ben Knoble defended its utility for power users.
   - **ODB abstraction**: Patrick Steinhardt’s 11-patch series (2026/07/07/15-32-32) to make ODB optimizations pluggable saw **initial review** from Junio Hamano, who endorsed the design goals (extending the "pre-auto-gc" hook and reordering maintenance tasks) while raising minor points about test brittleness and hook logging.
   - **Rebase vs. cherry-pick**: A proposal to add a `-x` flag to `git rebase -i` (2026/07/06/13-35-54) to append "cherry picked from" trailers faced **substantive pushback** from Junio, Jeff King, and Phillip Wood. The critics argued the feature misaligns with `rebase`’s core semantics (moving history) and suggested extending `git cherry-pick` with an interactive mode (`-i`) instead. The thread now hinges on whether the author (Trevor Gross) will pivot to the `cherry-pick -i` approach or abandon the effort.

---

## Notable threads

### 1. **Commit-graph regression fix lands** (2026/07/07/09-59-41)
**Headline**: Split commit-graph chains now correctly propagate topological levels, fixing a **17x performance regression** in incremental writes.
**What happened**: Kristofer Karlsson’s two-patch series (v1) addressed a bug introduced in September 2025 (`199d452758`) where the `topo_levels` slab was only propagated to the topmost layer of a split commit-graph chain. This forced `compute_reachable_generation_numbers()` to re-walk the entire ancestry of commits parsed from lower layers, degrading performance from **233ms to 4133ms** for incremental writes on large repositories.
**Key details**:
- **Patch 1/2**: Added trace2 instrumentation to expose the DFS cost (7 steps vs. 1 step expected).
- **Patch 2/2**: Fixed the loop variable in the propagation logic (`chain->topo_levels` instead of `g->topo_levels`).
- **Test coverage**: Updated `t/t5324-split-commit-graph.sh` to verify the fix (flipped from `test_expect_failure` to `test_expect_success`).
- **Review**: Taylor Blau approved the fix as "obviously good" but suggested an architectural refactor (moving `topo_levels` to `write_commit_graph_ctx`) as a follow-up. Junio Hamano marked the series as "Expecting a reroll" for a commit-message correction (date misstatement: April 2025 → September 2025).
**Status**: **Ready for merge** once the commit-message nit is addressed. The fix is uncontested and targets users with `fetch.writeCommitGraph` enabled or running `git maintenance` after fetch.

---

### 2. **`git_hash_*()` API hardening series approved** (2026/07/07/04-55-56)
**Headline**: Jeff King’s 7-patch series makes `git_hash_discard()` idempotent and adds defensive runtime checks, aligning Git’s hash API with industry standards.
**What happened**: The series addresses inconsistent and unsafe usage of the `git_hash_*()` API, particularly the non-idempotent behavior of `git_hash_discard()`, which could leave the hash context in an inconsistent state if called twice. The patches:
1. Standardized all callers to use wrapper functions (patches 1–3).
2. Made `git_hash_discard()` idempotent by adding an `active` flag to `git_hash_ctx` (patch 4).
3. Simplified callers to rely on the new behavior (patches 5–6).
4. Added defensive runtime checks to all wrapper functions (patch 7).
**Key details**:
- **Files touched**: 33 files, including `hash.c`, `http.c`, `object-file.c`, and Coccinelle tooling (`tools/coccinelle/hash.cocci`).
- **New symbols**: `git_hash_ctx.active` flag, `git_hash_discard()` (now idempotent), and runtime checks in `git_hash_clone()`, `git_hash_update()`, etc.
- **Behavior**: No user-visible changes; the series is purely internal hardening.
- **Review**: Junio Hamano raised a design question about the asymmetry between `git_hash_discard()` (idempotent) and `git_hash_final()` (triggers `BUG()` on misuse), but Jeff King clarified this aligns with industry-standard practices (e.g., Rust’s ownership model). Brian M. Carlson provided authoritative technical context on why hash libraries treat multiple `final()` calls as bugs (length-extension attacks). Patrick Steinhardt and Junio approved the series after cosmetic fixes (e.g., unnecessary braces, typos).
**Status**: **Approved for merge** (v2 forthcoming with cosmetic fixes). The series is a one-time cleanup with no ongoing maintenance burden.

---

### 3. **ODB optimizations pluggable: initial review** (2026/07/07/15-32-32)
**Headline**: Patrick Steinhardt’s 11-patch series refactors ODB optimization logic to enable backend-specific housekeeping, a key step in the ODB abstraction effort.
**What happened**: The series prepares the codebase for alternative ODB backends (e.g., reftable, custom storage) by extracting existing optimization logic (repacking, geometric repacking, auto-gc) into a modular system. Key patches:
1. Extended the "pre-auto-gc" hook to guard all automated maintenance tasks (patch 1/11).
2. Reordered maintenance tasks to group ODB-related optimizations (patch 2/11).
3. Extracted repack/prune logic into `maintenance_task_odb()` (patch 3/11).
4. Eliminated global state in repack argument assembly (patch 4/11).
5. Relocated backend-specific configuration reads to call sites (patch 5/11).
6. Introduced `struct odb_optimize_options` to encapsulate optimization settings (patch 6/11).
7. Unified incremental and geometric repacking strategies under `odb_optimize()` (patch 7/11).
8. Centralized auto-condition checks in `odb_optimize_required()` (patch 8/11).
9. Parameterized ODB optimization helpers to remove `the_repository` reliance (patch 9/11).
10. Fixed signedness issues to avoid compiler warnings (patch 10/11).
11. Relocated `odb_optimize()` and `odb_optimize_required()` to the "files" ODB source backend (patch 11/11).
**Key details**:
- **Files touched**: `builtin/gc.c` (heavily refactored), `odb.c`, `odb.h`, `odb/source-files.c` (new), and `t/t7900-maintenance.sh` (143 lines added for hook tests).
- **New symbols**: `enum odb_optimize_strategy`, `struct odb_optimize_options`, `optimize` and `optimize_required` vtable callbacks.
- **Behavior**: No user-visible changes; the series is preparatory for pluggable backends.
- **Review**: Junio Hamano reviewed patches 1–3, endorsing the design goals (hook expansion, task reordering) while raising minor points about test brittleness and hook logging. No objections to merging; the series is under active review.
**Status**: **Under review** (patches 4–11 awaiting feedback). The series is a foundational step for the ODB abstraction effort, enabling future backends to define their own optimization strategies.

---

### 4. **Rebase vs. cherry-pick: design debate** (2026/07/06/13-35-54)
**Headline**: A proposal to add a `-x` flag to `git rebase -i` to append "cherry picked from" trailers faces **substantive pushback** from Junio, Jeff King, and Phillip Wood.
**What happened**: Trevor Gross proposed a `-x` flag for `pick`, `reword`, and `edit` commands in `git rebase -i` to append "(cherry picked from commit ...)" trailers, mirroring `git cherry-pick -x`. The feature aims to streamline workflows where users cherry-pick commits during interactive rebases but lack a native way to record commit origin.
**Key objections**:
- **Junio Hamano**: The `-x` flag’s purpose is undermined in a rebase workflow where original commits are discarded, making the annotation’s utility questionable. Endorsed Jeff King’s and Phillip Wood’s critiques.
- **Jeff King**: Suggested extending `git cherry-pick` with an interactive mode (`-i`) instead, leveraging its existing sequencer backend. Demonstrated that `cherry-pick` already generates a todo file (`.git/sequencer/todo`), implying minimal work to add interactivity. Noted that `rebase.missingCommitsCheck` is the primary rebase-specific hurdle.
- **Phillip Wood**: Endorsed Jeff’s critique and pressed Trevor to justify the rebase-centric approach. Highlighted edge cases (e.g., `edit -x` followed by `fixup`) where the trailer’s meaning becomes ambiguous. Suggested harmonizing global and per-commit `-x` flags by having `cherry-pick -x` automatically add `-x` to each todo line.
**Key details**:
- **Patch**: v1 introduced the `-x` flag, parsed as an argument (e.g., `pick 123456 -x`), and reused `TODO_RECORD_ORIGIN` logic. Fast-forwards skip the message (documented difference from `git cherry-pick -x`).
- **Test coverage**: Verified flag works with all three commands and checks output format.
- **Edge cases**: Struct initialization fix in `do_pick_commit()`; error handling for malformed todo lines (e.g., `pick -xabcdef 123456`) unresolved.
- **Alternative**: Jeff’s `cherry-pick -i` proposal could obviate the need for Trevor’s patch but requires significant design discussion.
**Status**: **Stalled** pending Trevor’s response to the design objections. The thread’s future hinges on whether he pivots to the `cherry-pick -i` approach or abandons the effort. Junio’s endorsement of the `cherry-pick -i` direction makes it the most likely path forward.

---

### 5. **Rust integration: runtime-configurable nanosecond timestamps** (2026/07/07/04-38-50)
**Headline**: A proposal to make nanosecond timestamp support runtime-configurable (`core.useNsec`) gains traction, with Jeff King and Patrick Steinhardt endorsing the approach over the existing build-time knob (`USE_NSEC`).
**What happened**: D. Ben Knoble’s patch to add a `nanosec` Meson build option (mirroring Autotools’ `USE_NSEC`) evolved into a broader discussion about the knob’s necessity. Jeff King and Patrick Steinhardt proposed:
1. **Always compile nanosecond support** into Git when the platform allows it.
2. **Expose the feature via a runtime config knob** (`core.useNsec`).
3. **Deprecate and eventually remove the `USE_NSEC` build knob** once the runtime config is available.
**Key details**:
- **Motivation**: The existing build-time knob (`USE_NSEC`) is rarely used and adds complexity. Runtime configuration makes the feature more accessible without requiring users to rebuild Git.
- **Compatibility**: The `USE_NSEC` knob will be retained as a transitional measure but deprecated in favor of `core.useNsec`.
- **Default behavior**: The runtime config knob’s default remains unresolved (Peff suggested `false` for interoperability; Brian M. Carlson proposed `true` on platforms where testing showed no issues).
- **Implementation**: The runtime config would be the primary mechanism for enabling nanosecond timestamps, with the build-time knob retained only to tweak the unconfigured default.
**Status**: **Design consensus reached**; awaiting a patch implementing `core.useNsec`. The discussion resolved the long-standing question of whether the build-time knob should be retained, with the consensus favoring deprecation.

---

## In brief

- **`git replay --linearize` v7 posted** (2026/07/07/19-07-24): Toon Claes’s series to flatten merge commits into a linear history received **final approval** after addressing Junio’s edge-case concern about combining `--linearize` and `--revert`. The series is now **ready for merge**.
- **`git history squash` v7 feedback** (2026/07/07/07-51-33): Harald Nordgren’s series to fold commit ranges into a single commit saw **substantive review** from Phillip Wood, who identified a **critical input validation gap** (failing to sanitize `rev-list` options passed after `--`). A v8 is expected to address this and other usability concerns (e.g., template reordering for autosquash markers).
- **Reftable tombstone performance fix** (2026/07/07/15-24-38): Kristofer Karlsson’s two-patch series to fix a quadratic performance regression in the reftable backend received **review feedback** from Patrick Steinhardt, who identified test design issues (measurement isolation, tombstone carryover). The series is **under revision** but uncontested.
- **`includeIf.worktree` symlink handling resolved** (2026/07/07/15-26-19): Patrick Steinhardt endorsed Chen Linxuan’s proposal to store a non-realpath worktree path in `struct repository`, aligning `worktree` and `gitdir` behavior. Junio Hamano signaled **readiness to merge v7** once submitted.
- **`git cat-file --batch-command` v15 nears merge** (2026/07/07/08-50-41): Pablo Sabater’s GSoC series to add a `remote-object-info` command for querying object metadata from remotes addressed **all review feedback** except a **critical refactoring flaw** in patch 5/13 (uninitialized local variable and lingering global variable). Junio Hamano proposed a fix (initialize the local variable to `0` and split the `hash_algo` type change into a separate patch). The series is **ready for merge** once the flaw is addressed.
- **`git log --graph` cascading indentation v7** (2026/07/07/06-31-35): Pablo Sabater’s series to implement cascading indentation for visual roots in `git log --graph` received **final review feedback** from Chandra Pratap (cosmetic and defensive improvements). The series is **ready for merge** once the minor updates are incorporated.
- **Config lock timeout feature stalled** (2026/07/07/11-39-07): Johannes Schindelin’s nudge to merge Jörg Thalheim’s `core.configLockTimeout` patch was met with Junio’s reminder that the series remains **stalled** pending Patrick Steinhardt’s response to a technical question about caching behavior. The patch is otherwise ready for merge.
- **Rustification build system adjustments for macOS** (2026/07/07/16-51-12): Shardul Natu and Koji Nakamaru’s series to add Universal Binary support for Rust components on macOS was **queued in Junio’s integration branches** (v6). The series is **under review** and likely to graduate to `next`.
- **ANSI SGR sequence handling in sideband** (2026/07/07/11-45-44): Jeff King’s patch to accept colons (`:`) as sub-parameter separators in ANSI SGR escape sequences received **final approval** from Johannes Schindelin. The patch is **ready for merge**.
- **HTTP authentication bugfix** (2026/07/07/19-16-58): Junio Hamano confirmed that Aaron Plattner’s fix for preserving WWW-Authenticate headers during redirects is **merged to `master`**, but the proposed architectural refactoring (`credential_update_url()` helper) is **stalled** due to lack of reviewer attention.
- **`git history reword` feedback requirements** (2026/07/07/05-09-34): Dominique Martinet and D. Ben Knoble identified **critical UX failure modes** in Pablo Sabater’s `git history reword` series, including silent success for unreachable commits and lack of feedback showing old/new commit hashes. The discussion expanded to include **ref reachability verification** and **strict vs. permissive modes**.
- **Quadratic-time index scan fix** (2026/07/07/21-01-45): Henrique Ferreiro’s patch to fix a quadratic-time index scan in `next_cache_entry()` received **critical review** from Junio Hamano, who raised a **correctness concern** about the interaction between the new `cache_bottom` optimization and existing `.cache_bottom` logic (designed to handle index/tree ordering mismatches). The patch is **under revision**.
- **Unexpected full download during `git fetch`** (2026/07/07/22-32-22): SZEDER Gábor reported a bug where `git fetch` downloaded **300 MiB of objects** despite only ~1,400 being new. Junio Hamano suggested ruling out GitHub’s server-side behavior as a first step, but the root cause remains **unidentified**.

---

## On the radar

- **`git history squash` v8**: Harald Nordgren’s series to fold commit ranges into a single commit will need to address Phillip Wood’s feedback on **input validation** (sanitizing `rev-list` options) and **template reordering** for autosquash markers.
- **`cherry-pick -i`**: Jeff King’s proposal to extend `git cherry-pick` with an interactive mode (`-i`) is the most likely path forward for the `rebase -i -x` feature, but requires design discussion and implementation.
- **`core.useNsec`**: The runtime-configurable nanosecond timestamp feature awaits a patch implementing the new config knob.
- **ODB abstraction follow-ups**: Patrick Steinhardt’s pluggable ODB optimizations series will likely see further review of patches 4–11, with a focus on edge cases in memory estimation and pack selection.