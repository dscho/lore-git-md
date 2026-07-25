# The Git Mailing List Daily Digest for 2026/07/24

## The day in brief

A busy day on the Git mailing list with **87 emails across 20 threads**, dominated by two major feature series reaching completion and several important bugfixes. The **`git cat-file --batch-command` remote-object-info series** (13 patches) and **`git branch --delete-merged` series** (7 patches) both reached technical completion and are now queued for `next`, marking significant progress in object metadata querying and branch management workflows. A **packfile URI race condition fix** (3 patches) also advanced, though a test hang remains under investigation. The day saw substantive architectural discussions about **ODB alternates performance** and **Coccinelle CI toolchain limitations**, with maintainer decisions pending on long-term directions.

---

## Notable threads

### **Remote object metadata querying lands in `next`**
**Subject:** [PATCH v21 00/13] GSoC: `git cat-file --batch-command` remote-object-info

The **13-patch series adding `remote-object-info` to `git cat-file --batch-command`** reached its 21st iteration and is now **technically complete and queued for `next`**. This feature allows Git clients to query object metadata (currently size, with type support designed for future expansion) from remote repositories via the `object-info` protocol v2 capability, without downloading full objects.

Key technical details:
- **Dynamic capability negotiation**: Clients adapt to server-advertised attributes (e.g., `"size"`, `"type"`), returning empty strings for unsupported fields.
- **Robust input validation**: 10,000-object batch limit, 8KB URL length cap, strict OID format checks.
- **Memory safety**: Uses `xcalloc` and `free_object_info_contents()` to prevent leaks.
- **Test coverage**: 747 lines of new tests covering all smart transports (`git://`, `file://`, `http://`), `--buffer` mode, `-Z` output, and edge cases.

The series is **authored primarily by Pablo Sabater, Eric Ju, and Calvin Wan**, with sign-offs from Jonathan Tan and Christian Couder. Junio C Hamano’s **merge confirmation** marks it as ready for integration, with only cosmetic fixes (documentation cleanup, macro formatting) remaining. This is a **major GSoC milestone** and a significant step toward efficient object metadata querying in distributed workflows.

---

### **Safe automated branch cleanup reaches completion**
**Subject:** [PATCH v22 00/07] `git branch --delete-merged`

Harald Nordgren’s **7-patch series introducing `git branch --delete-merged`** reached its 22nd iteration and is now **implementation-complete, unblocked, and ready for merging**. The command provides safe, automated cleanup of local branches whose work has already landed on their upstream, with comprehensive safety controls:

- **Order-independent stacked-branch protection**: Fails to delete a branch if it is used as an upstream for an unmerged branch, but clears the upstream config for kept (merged) branches whose own upstream is being deleted. Implemented via a **single-pass graph traversal** that collects all protected branches before mutating the deletion set.
- **Per-branch opt-out**: `branch.<name>.deleteMerged=false` exempts specific branches from automated cleanup while allowing explicit deletion via `git branch -d`.
- **`--dry-run` preview**: Prints "Would delete" or "Would skip" messages without modifying refs or config.
- **Repeatable upstream selectors**: Supports multiple `--delete-merged` arguments (e.g., `--delete-merged origin/main --delete-merged origin/next`) and optional positional patterns (e.g., `topic-*`).

The series is **well-tested**, with 200+ lines of new tests covering edge cases like self-referential upstreams, missing upstreams, and ref iteration order independence. Junio C Hamano has **explicitly confirmed there are no outstanding issues**, and the only follow-up work is extending stacked-branch protection to `git branch -d` for consistency. This is a **major workflow improvement** for users managing stacked branches or topic-branch workflows.

---

### **Packfile URI race condition fix advances**
**Subject:** [PATCH v4 0/3] Fix packfile URI download race conditions

Ted Nyman’s **3-patch series fixing race conditions in Git’s HTTP transport** during concurrent packfile URI and dumb HTTP downloads advanced to its fourth iteration. The core issue: when multiple Git processes fetch the same packfile into a single object database, they can corrupt the temporary pack file by appending duplicate data. The solution uses **deterministic staging paths opened in read-write mode (without append)**, ensuring overlapping responses write identical bytes at identical offsets.

Key changes in v4:
- **Test robustness improvements**: The overlapping-download test server now includes a 60-second timeout, notifies FIFO waiters on failure, and tracks the server process for reliable cleanup.
- **HTTP 416 trace matching**: Now accounts for both older and current libcurl output formats.

Junio C Hamano reported a **test hang in `t5550-http-fetch-dumb.sh`** after merging the v3 series into `seen`, but the v4 changes appear to resolve this. The series is **rebased onto current `master` and ready for review**, with no open technical questions. This is a **critical bugfix** for large-scale hosting environments (e.g., GitHub, GitLab) that rely on packfile URIs for efficient object distribution.

---

### **ODB alternates performance regression sparks architectural discussion**
**Subject:** Performance regression in `git receive-pack` connectivity check after ODB rework

A **performance regression in server-side Git operations** (12% slowdown on fast filesystems; worse on NFS) introduced by the ODB abstraction rework (`a593373b09`) sparked a substantive architectural discussion. The regression manifests as excessive failing `open()` calls for loose objects across multiple ODB backends (quarantine and main object stores, alternates), particularly costly on NFS-backed deployments.

Patrick Steinhardt proposed a **fundamental architectural shift**: moving alternates handling into the "files" backend itself, treating alternates as an implementation detail rather than a cross-backend concern. This would enable:
- **Pack-first-loose-second lookups across all alternates**, eliminating redundant loose object checks.
- **Simplified commit-graph/bitmap loading** by dropping the `OBJECT_INFO_SECOND_READ` flag.
- **Cleaner object storage extension design**.

The proposal directly addresses the root cause of the regression (the ODB rework’s shift to "pack-loose, pack-loose" lookups across backends) and resolves secondary issues like cross-process coordination. Patrick plans to implement this in stages, starting with a patch series to remove `odb_prepare_alternates()` calls (delayed until late July). The discussion remains open on whether a **near-term fix** (e.g., targeted lookup order adjustment) will be implemented before the architectural redesign.

---

### **Coccinelle CI toolchain limitations debated**
**Subject:** [PATCH] builtin/branch.c: avoid Coccinelle 1.1.1 performance regression

A **Coccinelle 1.1.1 performance regression** in the `separate_loop_index` rule triggered a discussion about whether to **upgrade the CI toolchain** or **work around toolchain limitations in the codebase**. The regression caused the static-analysis job on the `seen` integration branch to hit its six-hour timeout due to reusing a single loop index variable declared at function scope across two separate loops in `delete_merged_branches()`.

The **pragmatic workaround** (declaring each loop index in its respective `for` loop) was **merged to `master`**, reducing the Coccinelle check from a timeout to 0.06 seconds. However, Jeff King (Peff) and Junio C Hamano both **prefer addressing the root cause** by upgrading the CI toolchain to Coccinelle 1.3.0+ (which includes a fix for the exponential slowdown). The discussion remains open, with Junio recommending a **two-phase approach**:
1. Apply the immediate, low-risk fix (changing the second loop index from `size_t` to `int` to match `candidates.nr`).
2. Defer the broader standardization of ref-counting types to `size_t` for a future, quieter moment.

---

## In brief

- **`git rebase --update-refs` symref bugfix**: Son Luong Ngoc’s **2-patch series** fixing edge cases with branch symrefs in `git rebase --update-refs` received substantive review from Phillip Wood, who identified a potential bug in HEAD comparison logic and reiterated requests for an explanatory comment and minor test organization tweaks. The series is **functionally complete and under active review**.

- **GSoC: Partial clone disk space recovery**: Siddharth Shrimali’s **progress report** on improving disk space recovery for partial clones refined the design for the `git maintenance run --task=gc-promisor` subcommand. The work remains in the prototyping phase, with performance testing on large promisor packs underway.

- **`git show-branch` slab conversion**: Gatla Vishweshwar Reddy’s **refactoring patch** converting `git show-branch` to use a commit-slab instead of `object.flags` is **technically ready for integration** pending resolution of minor mechanical cleanups (indentation, variable scoping, test coverage). Junio C Hamano has not yet commented on the v10 patch.

- **`git diff --relative --cached` segfault fix**: Jeff King’s **bugfix patch** for a segfault in `git diff --relative --cached` when operating on unmerged paths outside the current directory was **reviewed and accepted by Junio C Hamano**. The fix adds a NULL check before `fill_filespec()` and includes a regression test.

- **`git repo info` path-discovery keys**: K Jayatheerth’s **7-patch series** adding path-related keys to `git repo info` resolved its last behavioral ambiguity. Junio C Hamano clarified that `path.index` should return the path held by Git’s internal `repo->index_file` member, regardless of whether the repository is bare or the file exists. The series is now **ready for deeper technical review**.

- **`git config --url` scp-style support**: Fabian Pottbäcker’s **3-patch series** adding scp-style SSH URL support to `git config --url` received surface-level review from Ben Knoble, who identified a copy-paste error in comments. The series is **under review and uncontroversial**.

- **ODB on-disk structures pluggability**: Patrick Steinhardt’s **5-patch series** making on-disk ODB structures pluggable received substantive review from Junio C Hamano. The second patch’s unconditional downcast in the loop iterating over ODB sources was questioned, with Junio suggesting a more backend-agnostic approach (adding a `load object map` method to the `odb_source` interface). The series is **under review and part of the ongoing ODB abstraction effort**.

- **Packfile duplicate object hardening**: Taylor Blau’s **5-patch series** hardening the packfile subsystem against duplicate object entries received an **implicit Ack from Junio C Hamano**, who clarified that packs with duplicate entries are **invalid** and emphasized the importance of early detection. The series is **under initial review**.

---

## On the radar

- **`git rebase -i` fixup/squash edge cases**: Phillip Wood’s **2-patch series** fixing edge cases in `git rebase -i` (incorrect commit count in template messages and editor not opening after `fixup -c` following a skip) received substantive review from Junio C Hamano. The first patch’s reading logic must handle trailing newlines in legacy command-list files, and the second patch’s string matching could be tightened to require a space after the verb. The series is **likely to proceed to `next` once the feedback is addressed**.

- **macOS fsmonitor memory leak**: Tamir Duberstein’s **bugfix patch** for a memory leak in Git’s macOS fsmonitor daemon received tested review from Koji Nakamaru, who suggested improving the commit message with additional benchmark data. The patch is **under review and low-risk**.

- **Submodule URL alias fix**: Éric NICOLAS’s **bugfix patch** for `git submodule update --remote` failing to match submodule remotes when URL aliases are in use was **merged to `master`**. The fix ensures the submodule’s configured remote is matched correctly by resolving aliases before comparison.

- **macOS TRE regexec() memory leak**: Chungmin Lee’s **platform-specific bugfix** for a memory leak in Git’s macOS builds when the system regex engine processes invalid UTF-8 sequences received structural feedback from Junio C Hamano. The discussion remains open on the **long-term maintainability** of the workaround and the **organization of platform-specific overrides**.

- **`git clone --revision` segfault fix**: Adrian Friedli’s **bugfix patch** for a segfault in `git clone --revision` when the server advertises more refs than requested was **merged to `master`** after adding a regression test. The fix adds a NULL check in `update_remote_refs()`.

- **`git receive-pack` connectivity check regression**: The **performance regression in `git receive-pack` connectivity checks** after the ODB rework remains under discussion. Patrick Steinhardt’s proposal to move alternates handling into the "files" backend is the leading architectural direction, but a **near-term fix** may still be needed to address the immediate performance impact.