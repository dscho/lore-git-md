## The day in brief

Saturday, August 1, 2026 was a **busy but focused** day on the Git mailing list, with **51 emails across 12 threads**. The standout developments:

- **Michael Montalbo’s `diff-provider` RFC v7** posted, proposing a unified abstraction layer for diff hunks that could unify `git blame`, `--stat`, and external diff tools under a single interface.
- **Harald Nordgren’s `git history squash` v11** landed, functionally complete and ready for integration, resolving the last usability debate by making message editing the default.
- **Ted Nyman’s packfile URI race-condition fix v6** received **final approval from Peff**, clearing the last technical hurdle before merge.
- **Junio’s `git add --resolved` v3** saw **usability debate** with Peff, but the series remains on track for `next`.

The day leaned **technical and integrative**—multiple long-running efforts reached key milestones, while new RFCs and design discussions kept the pipeline full.

---

## Notable threads

### **RFC: diff provider interface (v7)**
**Headline**: Michael Montalbo posted the **seventh revision** of the hunk provider interface RFC, now restructured as a **unified abstraction layer** (`diff-provider.h`) that lets Git consult alternate sources for diff hunks before falling back to xdiff. The series ships two example providers: an **in-process cache** (`diff-hunks` store) and an **external process driver** (`diff.<driver>.process`), both answering by object identity alone.

### What’s new in v7

- **Unified framework**: The interface (`diff-provider.h`) establishes a repository-owned chain of providers, each queried in turn until one answers or fails, with xdiff as the terminal provider. The chain is built on first consultation and released in `repo_clear()`, ensuring provider state (e.g., a running process) never outlives its repository.
- **Performance validated**: Default path (no configured providers) runs within noise of pre-series code, confirming the abstraction adds no measurable overhead. The `diff-hunks` store delivers **1.26–1.9× speedups** for warmed caches.
- **Safety and validation**: Hunks must pass `diff_provider_check_hunk()` (bounds, overlap, alignment checks) before consumers see them. Graceful fallback for malformed responses, transient errors, and permanent withdrawal.
- **Documentation**: New man pages (`git-diff-provider(7)`, `gitformat-diff-provider(5)`) explain the interface, store format, and protocol. A patch clarifies the limitations of traditional external diff drivers, underscoring the need for the new abstraction.

### Key technical details

- **Files touched**: `diff-provider.c`, `diff-hunks.c`, `diff-process.c`, `diff.c`, `blame.c`, `sub-process.c`, plus documentation and tests.
- **New symbols**: `diff_provider_consult_fn`, `diff_provider_emit_fn`, `diff_request`, `diff_provider`, `repo_diff_providers`, `diff_provider_check_hunk()`, `enum diff_provider_outcome` (`answered`, `unanswered`, `failed`, `refused`).
- **Store design**: The `diff-hunks` store is a chunked binary file at `$GIT_DIR/objects/info/diff-hunks`, keyed by `(old_blob, new_blob, xdl_opts)`. Entries are recorded only when trimmed/untrimmed diffs agree, serving both blame (zero context) and stat formats (any `-U`). Writing is opt-in (`GIT_DIFF_HUNKS_WRITE=1` or `diffHunks.write`).
- **External process protocol**: Versioned pkt-line protocol (version=1, capability=hunks) over stdin/stdout. Requests send only object names (`old-oid`/`new-oid`), enabling providers to answer before any blob is read. The process is authoritative: its answers may differ from xdiff and override the `diff-hunks` store.

### Open questions

- **Design feedback**: The RFC explicitly asks whether the direction is right, not for the 10 patches to merge as one. If accepted, the work will return as separate series: (1) interface and `diff-hunks` store, (2) external process provider, (3) content enrichment (content-carrying requests and patch output integration).
- **Scope**: The series focuses on identity-keyed requests; content-carrying requests (for patch output) are deferred. The external process provider currently supports only `git blame` and summary formats (`--stat`, `--numstat`, `--shortstat`); patch output and `git log -L` remain builtin.

**Status**: **Technically complete for its scope**, awaiting design feedback. All substantive review feedback from v6 has been addressed, including the merge conflict with `mm/line-log-limited-ops`, documentation clarifications, and subprocess API refinements.

---

### **`git history squash` v11**
**Headline**: Harald Nordgren posted the **eleventh iteration** of the `git history squash` series, now **functionally complete** and making **message editing the default behavior**. The series is ready for integration, with Junio’s prior "Will replace" sign-off on v7 and all technical feedback resolved.

### What’s new in v11

- **Default message editing**: The command now opens an editor with the autosquash-style template by default, matching `git rebase -i --autosquash`. The `--no-edit` flag skips the editor, keeping the oldest commit’s message (or the last `amend!` targeting it). This resolves the usability debate from v10, where Phillip Wood argued for encouraging commit hygiene.
- **Stricter range validation**: Rejects ranges that reach a root commit or have more than one tip, ensuring the selected history has exactly one boundary and one tip.
- **Ref-handling refinement**: Only local branches pointing inside the range are protected by default; tags and remote-tracking refs are left unchanged.
- **Patch consolidation**: The sequencer preparation and message-editing logic are now folded into a single patch (4/4), streamlining the implementation.

### Key technical details

- **Files touched**: `builtin/history.c`, `sequencer.c`/`sequencer.h`, `advice.c`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`.
- **New helpers**: `first_parent_tree_oid()` (refactored), `commit_tree_ext()` with `message_template` parameter, extracted sequencer helpers (`add_squash_combination_header`, `add_squash_message_header`, `squash_subject_comment_len`, `TODO_REPLACE_FIXUP_MSG`).
- **Behavior**: Folds a commit range into its oldest commit while preserving descendant history. Rejects operations with local branches pointing inside the range by default, with advice guiding users to `--update-refs[=head]`. Handles interior merges, `fixup!`/`squash!`/`amend!` commits, and rev-list option validation.
- **Test coverage**: 776 lines in `t/t3455-history-squash.sh`, covering edge cases like interior merges, ref handling, and marker grouping.

**Status**: **Ready for integration**. Junio’s "Will replace" sign-off on v7, the resolution of all feedback in v10 and v11, and Matt Hunter’s explicit v10 sign-off signal intent to merge. The series is **technically sound** and awaits Junio’s next "What's cooking" report for queuing in `next`.

---

### **Packfile URI race-condition fix (v6)**
**Headline**: Ted Nyman’s **v6 packfile URI race-condition fix** received **final approval from Peff**, clearing the last technical hurdle before merge. The series is now **fully approved** by all key reviewers (Junio, Taylor Blau, Peff) and in a "ready for merge" state.

### What’s fixed

- **Race conditions in HTTP transport**: Concurrent processes fetching the same packfile URI or dumb HTTP pack into a single object database no longer corrupt data. The fix uses **deterministic staging paths** and **read-write mode without append**, ensuring concurrent processes write identical bytes at identical offsets.
- **Windows/MINGW compatibility**: Handles sharing semantics (`O_EXCL` loop workaround) and `.keep` file edge cases.
- **HTTP 416 handling**: Gracefully handles completed partial packs, with Peff’s forward-compatibility note about remote file changes mid-transfer (not a regression).

### Key technical details

- **Files touched**: `http.c`, `fetch-pack.c`, `t/t5550-http-fetch-dumb.sh`, `t/t5702-protocol-v2.sh`.
- **Behavior**: Preserves resumable downloads for both packfile URIs and dumb HTTP. Sets `delete_redundant = 1` to remove old promisor packs containing dropped objects, **implying `-d`**.
- **Test coverage**: 240 stress-test runs of the overlapping-download scenario, plus regression tests for HTTP 416 handling and `.keep` file edge cases.

**Status**: **Ready for merge**. Junio intends to graduate the series to `next` once queued. No further rerolls are expected.

---

### **`git add --resolved` v3**
**Headline**: Junio’s `git add --resolved` series saw **usability debate** with Peff, who raised concerns about the feature potentially creating a **false sense of completeness** by staging only files with resolved conflict markers while ignoring related changes (e.g., updates to callers of a modified function). Junio countered that Git’s existing merge behavior (refusing to operate on paths with local modifications) mitigates the risk, citing test case `t6423` ("Testcase 11b") as validation.

### Key technical details

- **New option**: `git add --resolved` stages only paths whose conflict markers have been removed.
- **New helpers**: `is_conflict_marker_line()` (consolidates duplicate logic in `diff.c` and `rerere.c`), `remove_file_from_index_with_flags()` (supports `--dry-run` and `--verbose`), `has_conflict_markers()` (scans file content for remaining markers, with early exit for binary files).
- **Files touched**: `merge-ll.c`, `read-cache.c`, `builtin/add.c`, `Documentation/git-add.txt`, `t/t2207-add-resolved.sh`.
- **Test coverage**: Covers refusal to stage files with markers, success when all markers are removed, ignoring unconflicted modified files, handling deletions, pathspecs, and option incompatibility with `-u` and `-A`.

### Open questions

- **Usability risk**: Peff’s concern about staging incomplete resolutions remains unresolved in terms of documentation or command output. The test suite does not currently cover scenarios where related changes exist outside conflicted files.
- **Design debate**: Michael Montalbo’s proposal to split `--resolved` into a more general `--unmerged` selector with explicit policy flags remains unaddressed.

**Status**: **Technically sound**, but the usability discussion may prompt minor documentation tweaks before integration into `next`.

---

## In brief

**`git bisect --reset-when-found` v4** -- Harald Nordgren’s series adding an auto-reset option to `git bisect` is **ready for `next`**, with all surface-level feedback addressed. Junio proposed a **refactoring** to simplify the internal state machine, but the series remains on track for integration.

**`git repack --drop-filtered` RFC v2** -- Siddharth Shrimali’s series for partial clones now **concedes the safety guards** (merge/rebase/cherry-pick checks and index validation) are **optional optimizations** rather than correctness measures. Junio’s feedback on the reliability of `ODB_FOR_EACH_OBJECT_PROMISOR_ONLY` and repack machinery behavior remains open.

**`git cat-file --batch-command` `%(objecttype)` v2** -- Pablo Sabater’s GSoC series extending remote-object-info to include object type metadata saw **substantive feedback from Peff**, who recommended **bypassing `struct object_info` entirely** for the remote path. The series is otherwise complete, with only minor nits left to address.

**Git 3.0: lowercase-only hex object IDs** -- brian m. carlson’s RFC to restrict hex object IDs to lowercase in Git 3.0 saw **design debate** with Junio invoking the **Robustness Principle** and Peff identifying an **edge case** (uppercase hex in object payloads creating parallel histories). Junio downplayed the edge case, but the broader question of whether to accept the breaking change remains unresolved.

**`git filter-branch` bugfix** -- Grant Moyer fixed a **long-standing inversion** in commit mapping logic with `--state-branch`, causing the map directory to be populated backwards. The patch is **self-contained** and includes a test.

**Git Rev News edition 137** -- Christian Couder called for contributions to the next edition of the community newsletter, with a **publication deadline of August 3, 2026**.

---

## On the radar

- **`fetch_if_missing` per-repository flag** -- Tian Yuchen’s patch to move the global `fetch_if_missing` variable into `struct repository` needs a **v3 to address repository-less invocations** (e.g., `git index-pack` outside a repository). Junio identified a behavior change where lazy fetches could be inadvertently enabled.
- **`git add --resolved` usability documentation** -- Peff’s feedback on the feature’s potential to mislead users into staging incomplete resolutions may prompt **documentation tweaks** or a warning in the command’s output. The test suite should also cover scenarios with related changes outside conflicted files.