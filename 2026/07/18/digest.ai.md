Here’s the digest for **2026/07/18**, covering the day’s most consequential developments in Git’s mailing list traffic:

---

### The day in brief
**Saturday, July 18, 2026** was a **high-volume, milestone-heavy day** (47 emails, 16 threads) dominated by **finalization of long-running series** and **critical regressions** in recently merged or near-merged topics. The standout events:
1. **`git cat-file --batch-command` remote-object-info** (v20) is **technically complete** and queued for `next`, with only cosmetic nits remaining.
2. **`git history squash`** (v9) is **ready for integration**, resolving all prior feedback and positioning itself as a foundational piece of Git’s future history-editing architecture.
3. **Two high-priority regressions** surfaced in the sequencer and `git history fixup`, threatening to delay graduation of otherwise complete series.
4. **Design debates** flared around `no-ref-delta` and `git repack --drop-filtered`, with maintainers probing the trade-offs between strict enforcement and protocol flexibility.

---

### Notable threads

#### **1. `git cat-file --batch-command` remote-object-info (v20) – Ready for `next`**
**Headline**: The 20th iteration of Pablo Sabater’s GSoC series is **technically complete**, addressing all prior feedback and queued for `next`. The series enables Git clients to query object metadata (size, with type support designed for future expansion) from remote repositories without downloading full objects, using the `object-info` protocol v2 capability.

**Key developments**:
- **Build fixes**: SZEDER Gábor and Pablo Sabater resolved a signed/unsigned comparison warning in `write_fetch_command_and_capabilities()` by changing `hash_algo` from `int` to `const unsigned int`. The fix is mechanical and must land before graduation to `master`.
- **Dynamic capability negotiation**: The final patch in v20 replaces the hardcoded allow-list for `object-info` capabilities with a runtime-negotiated one, enabling forward compatibility as the protocol expands. The client now silently omits unsupported placeholders (e.g., `%(objecttype)` when the server doesn’t advertise `type`), matching `for-each-ref` behavior.
- **Memory safety**: A leak in `transport-helper.c` (fixed in v19) is carried forward, ensuring `data->name` is freed unconditionally in `disconnect_helper()`.
- **Edge cases**: Bare OID requests now trigger unconditional existence checks on the server, closing a gap where such requests could silently succeed without validation. The check uses `OBJECT_INFO_SKIP_FETCH_OBJECT` and `OBJECT_INFO_QUICK` flags to avoid partial-clone fetches.
- **Test coverage**: 747 lines of new tests in `t/t1017-cat-file-remote-object-info.sh` verify all three smart transports (`git://`, `file://`, `http://`), `--buffer` mode, `-Z` output, and edge cases like malformed OIDs and unsupported placeholders.

**Status**: **Queued for `next`**. Junio’s prior confirmation that no unresolved design or implementation concerns remain still holds. The series is ready for final polish before merging to `master`.

**Files touched**: `builtin/cat-file.c`, `fetch-object-info.c`, `transport-helper.c`, `connect.c`, `serve.c`, `Documentation/`, `t/`.

---

#### **2. `git history squash` (v9) – Ready for integration**
**Headline**: Harald Nordgren’s `git history squash` series (v9) is **functionally complete** and ready for integration, resolving all prior feedback and positioning itself as a foundational piece of Git’s future history-editing architecture. The command efficiently collapses a commit range into its oldest commit while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach.

**Key developments**:
- **Autosquash alignment**: The `--reedit-message` template now **exactly matches `git rebase -i --autosquash`**, grouping `fixup!`/`squash!`/`amend!` commits under their targets and fully commenting out `fixup!` messages while retaining `squash!`/`amend!` bodies.
- **`amend!` message handling**: **Resolved** by incorporating `amend!` messages targeting the first folded commit into the default squashed commit message, addressing Matt Hunter’s consistency concern.
- **Rev-list option validation**: The command now sanitizes all `rev-list` options (including those after `--`) by forcing `--reverse --topo-order` and warning if the user’s options would alter the walk order.
- **Merge handling**: Interior merges are folded as long as the range has a single base, with `--ancestry-path` ensuring only commits descended from the base are included.
- **Documentation refinements**: Addressed Matt Hunter’s feedback by removing a redundant sentence about preserving commit effects and splitting the `--reedit-message` description into its own paragraph.
- **Test suite readability**: Improved with test helpers (`stage_file`, `commit_with_message`, `check_commit_count`, etc.), following the pattern established in the `delete-merged` topic.

**Architectural vision**: Junio C Hamano **fully endorses** D. Ben Knoble’s vision of using `git history squash` and `git replay` as a faster, non-interactive backend for `git rebase --autosquash`, calling it "an ideal future" and confirming that the ongoing `git history` work is already moving the project in this direction. The implication is that the eventual goal is to **replace the internals of `git rebase -i` entirely**, eliminating working-tree manipulation and enabling a fully modular, conflict-free history-editing pipeline.

**Status**: **Ready to queue**. Junio’s "Will replace" sign-off on v7 indicates intent to merge, and v9 addresses all remaining technical and documentation concerns. The series is now **technically sound** and awaits Junio’s next "What's cooking" report for formal queuing.

**Files touched**: `builtin/history.c`, `sequencer.c`/`sequencer.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.

---

#### **3. Sequencer regression: notes from dropped commits in interactive rebase**
**Headline**: A **high-priority regression** surfaced in Phillip Wood’s recently approved sequencer series: the fix for notes from dropped commits works in non-interactive rebase but **fails in interactive mode (`-i`)**. Uwe Kleine-König reported that notes from manually skipped commits (e.g., `git rebase --skip`) are still incorrectly copied to the next commit.

**Root cause**: The series only handles commits dropped by `--empty=drop` (non-interactive default), not user-initiated drops (e.g., `--skip` or `--continue` without committing). Interactive rebase defaults to `--empty=ask`, which behaves inconsistently (drops on `--continue`, keeps on `--skip`).

**Phillip’s follow-up**: Confirms the gap and proposes revising `--empty=ask` to keep empty commits on `--continue` and drop them on `--skip` for consistency. This would be a **breaking change** but aligns with the series’ goal of preventing note corruption.

**Status**: **Blocking for `next`**. The series is **not ready for `next`** until the interactive rebase backend is fixed. No patches have been proposed yet; Phillip is still designing the solution.

**Files implicated**: `sequencer.c`, interactive rebase backend.

---

#### **4. `git history fixup` corrupts index in other worktrees**
**Headline**: Toon Claes reported a **critical bug** in `git history fixup`: when run in one worktree, it leaves spurious staged changes in other worktrees that share the same repository, corrupting their index state. Phillip Wood identified the root cause: `git history fixup` rewrites commits pointed to by branches checked out in other worktrees without updating their index or working copy.

**Proposed solution**: Adopt `git rebase --update-refs` behavior—either refuse to rewrite branches checked out in other worktrees, or update their index and working copy if clean. Phillip also flags a missing safety mechanism to prevent concurrent rewrites of the same branch.

**Status**: **Bug report confirmed**; no patch yet. This is a **high-priority follow-up** for the `git history` series, as it exposes a fundamental flaw in worktree isolation.

**Files implicated**: `builtin/history.c`, `worktree.c`, `read-cache.c`.

---

#### **5. `no-ref-delta` capability: design debate reopens**
**Headline**: Jeff King (Peff) reopened the design debate around Taylor Blau’s `no-ref-delta` capability, probing whether the proposed strict enforcement (`OFS_DELTA`-only packs) is necessary for the write-through proxy use case. Peff suggests a narrower "bases-before-deltas" capability that enforces Git’s default ordering without banning `REF_DELTA` outright, arguing it could achieve similar performance goals with broader protocol compatibility.

**Taylor’s response**: Clarifies that the proxy’s asynchronous indexer relies on offset-based addressing and processes packs in a single forward pass. Even a backward `REF_DELTA` requires an OID lookup, complicating the proxy’s offset-based reconstruction metadata. The discussion leaves open whether the stricter `no-ref-delta` approach is justified by the proxy’s specific needs.

**Status**: **Design phase**. No consensus yet on the optimal solution. The thread remains focused on the trade-offs between strict enforcement and protocol flexibility.

**Files implicated**: `pack-objects.c`, protocol capabilities.

---

### In brief
- **`git repack --drop-filtered` (RFC v1)**: Christian Couder’s **surface-level review** of patch 1/7 endorsed the current CLI design (`--drop-filtered` + `--dry-run`) but requested a commit message rationale. The series remains blocked by a **logical flaw** in `--drop-filtered` vs `--write-bitmap-index` validation (identified by Junio).
- **`the_repository` removal (lockfile/tempfile APIs)**: René Scharfe clarified the selection criteria for converted callers (only those already holding a `struct repository *` reference) and pushed back on Patrick Steinhardt’s proposed compile-time guard for the old global functions, calling it a task for a **separate series**.
- **Trace2 recursion crash**: Derrick Stolee and Taylor Blau converged on a **two-part fix**: Taylor’s alternative (disabling `libintl_vsnprintf()` redirection for `xsnprintf()`) and Derrick’s patch (removing `xsnprintf()` from trace2 entirely).
- **`git bisect --auto-reset` (v2)**: Johannes Sixt raised a **platform-compatibility concern** about the refactoring in patch 1, questioning whether the commit message’s claim about avoiding file reopens is accurate on Windows. The series remains queued in `seen`.
- **`wt-status` performance refactoring (v3)**: Sahitya Chandra’s patch to replace O(n²) `string_list_insert()` calls with O(n log n) `string_list_append()` + sort is **fully reviewed and endorsed** by Jeff King and Patrick Steinhardt. Ready for integration.
- **ODB abstraction (loose backend)**: SZEDER Gábor identified a **build-breaking issue** in Patrick Steinhardt’s series due to a missing evil merge resolution. A v2 is required.
- **Swift userdiff driver**: Johannes Sixt’s **substantive review** identified test ambiguities (duplicate `RIGHT` tokens) and regex optimizations (e.g., enumerating Swift keywords explicitly). The patch remains queued in `seen` pending validation from Swift users.
- **`git last-modified` Bloom filter optimization (v1)**: Jeff King and Taylor Blau engaged in **substantive design discussions** on patches 3/4 and 4/4, focusing on the `--show-trees` correctness bug and maintainability of the `bloom_filter_settings` restoration. Taylor provided **tested code snippets** for two solutions to the `--show-trees` issue, with the simpler "remove uncovered paths" approach recommended.
- **`git branch -d` error message improvement**: Junio and René debated whether to simplify error messages by merging the `BRANCH_CHECKOUT_KIND_UPDATE_REF` case into "rebase" and whether to use `enum` instead of `int` for the checkout kind. No consensus yet; test coverage remains incomplete.

---

### On the radar
- **Interactive rebase regression**: Phillip Wood’s follow-up to Uwe Kleine-König’s report is **high-priority** for the sequencer series. The fix will likely involve extending `PICK_RESULT_DROPPED` logic to cover user-initiated drops.
- **`git history fixup` worktree bug**: Phillip’s proposed solution (adopting `git rebase --update-refs` behavior) is **high-priority** for the `git history` series. Expect a patch soon.
- **`no-ref-delta` design**: The thread remains open pending further discussion on the trade-offs between strict enforcement and protocol flexibility. Taylor’s next reply may clarify whether the proxy’s design truly requires `OFS_DELTA`-only packs.