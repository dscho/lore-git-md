# The Git Mailing List Daily Digest for 2026/07/24

## The day in brief

A busy day on the Git mailing list with **87 emails across 20 threads**, marking significant progress on several long-running efforts. The **`git cat-file --batch-command` remote-object-info series** reached its final form and was merged to `next`, while Harald Nordgren's **`git branch --delete-merged`** series cleared its last technical hurdles and is now ready for integration. Performance regressions in the ODB abstraction and HTTP transport received focused attention, with architectural proposals emerging. Smaller bugfixes and refactorings continued to land, and the first patches from Google Summer of Code projects began appearing for review.

---

## Notable threads

### **`git cat-file --batch-command` remote-object-info series graduates to `next`**
**Headline:** The 21st iteration of Pablo Sabater's security-hardened series enabling `git cat-file --batch-command remote-object-info` has been **merged to `next`** after Junio C Hamano's explicit confirmation. The series allows Git clients to query object metadata (currently size, with type support designed for future expansion) from remote repositories without downloading full objects, using the new `object-info` protocol v2 capability.

**What changed:** The v21 revision made three targeted cosmetic changes since v20: dropping an unnecessary documentation change, fixing macro formatting, and correcting a typo. These addressed the last remaining surface-level nits identified by Karthik Nayak. The series remains structurally unchanged from v20, which resolved all technical and design concerns, including dynamic capability negotiation, memory leak fixes, and server-side consistency.

**Why it matters:** This feature addresses a long-standing gap in Git's remote object inspection capabilities, particularly valuable for partial clones and large repository workflows. The implementation is robust, with 747 lines of new tests covering all smart transports (`git://`, `file://`, `http://`), edge cases, and security considerations like input validation (10,000-object batch limit, 8KB URL length cap).

**Next steps:** The series is now cooking in `next` and on track for integration into `master` in a future release. The only unresolved cosmetic nits (translation context for an error message and alphabetical ordering in `Makefile`) are non-blocking.

---

### **`git branch --delete-merged` series unblocked and ready for merging**
**Headline:** Harald Nordgren's **22nd iteration** of the `git branch --delete-merged` series is now **implementation-complete, unblocked, and ready for merging**. Junio C Hamano has explicitly confirmed there are no outstanding issues, signaling the series is "pretty much done."

**What changed:** The v22 revision made a single mechanical adjustment to loop variables (`int i` instead of `size_t i`) to avoid typecasts and prevent CI timeouts, with no functional impact. The v21 revision resolved the last correctness issue (order dependency in stacked-branch protection) via a **single-pass graph traversal** that collects all protected branches before mutating the deletion set.

### Key features:

- **Order-independent stacked-branch protection:** Fails to delete a branch if it is used as an upstream for an unmerged branch, but clears the upstream config for kept (merged) branches whose own upstream is being deleted. The new implementation ensures consistent results regardless of ref iteration order.
- **Per-branch opt-out:** The `branch.<name>.deleteMerged=false` config key exempts specific branches from automated cleanup while allowing explicit deletion via `git branch -d`.
- **`--dry-run` preview:** Prints "Would delete" or "Would skip" messages without modifying refs or config.
- **Repeatable upstream selectors:** Supports multiple `--delete-merged` arguments to widen the upstream match (e.g., `--delete-merged origin/main --delete-merged origin/next`) and optional positional patterns (e.g., `topic-*`) to limit deletion scope.

**Why it matters:** This feature addresses a real workflow pain point for users managing stacked branches or iterative topic-branch workflows, providing safe, automated cleanup with comprehensive safety controls. The implementation is thoroughly tested, with 200+ lines of new tests covering all edge cases, including self-referential upstreams, missing upstreams, and ref iteration order independence.

**Next steps:** The series is expected to appear in an upcoming "What's cooking" report and graduate to `next` soon. The only agreed follow-up work is extending stacked-branch protection to `git branch -d` for consistency, which Harald has committed to implementing post-merge.

---

### **ODB performance regression sparks architectural proposal**
**Headline:** Patrick Steinhardt proposed a **fundamental architectural shift** to address the ODB performance regression introduced by the ODB abstraction rework. The regression, reported by Wolfgang Kritzinger (Atlassian), manifests as a 12% slowdown in server-side Git operations (pushes and alternates-based operations) due to a change in object lookup order from "pack-first, loose-second" *per backend* to "pack-loose, pack-loose" *across backends*.

**The proposal:** Move alternates handling from the ODB layer into the "files" backend itself, treating alternates as an implementation detail rather than a cross-backend concern. This would enable optimizations like pack-first-loose-second lookups across all alternates while simplifying other subsystems (commit-graph, bitmap, maintenance). The proposal directly addresses the root cause and resolves secondary issues like dropping the `OBJECT_INFO_SECOND_READ` flag and unifying commit-graph/bitmap loading.

**Why it matters:** The regression is particularly impactful for large-scale hosting environments (e.g., GitHub, GitLab) that rely on `GIT_ALTERNATE_OBJECT_DIRECTORIES` for object sharing between forks. The performance impact is exacerbated on NFS-backed deployments due to expensive loose object checks.

**Next steps:** Patrick plans to implement this in stages, starting with a patch series to remove `odb_prepare_alternates()` calls (delayed until late July). A near-term fix (e.g., targeted lookup order adjustment) may still be needed to address the immediate performance regression. The proposal has not yet received feedback from other key stakeholders (e.g., Peff, Junio), but its architectural direction aligns with earlier suggestions to optimize alternates handling.

---

### **HTTP transport race conditions fixed**
**Headline:** Ted Nyman's v4 series fixing race conditions in Git's HTTP transport during concurrent packfile URI and dumb HTTP downloads has resolved its last technical hurdle. The series preserves resumable downloads for both packfile URIs and dumb HTTP by using deterministic staging paths opened in read-write mode (without append), ensuring overlapping responses write identical bytes at identical offsets.

**What changed:** The v4 reroll included only test robustness improvements since v3, addressing a test hang reported by Junio C Hamano. The overlapping-download test server now includes a 60-second timeout, notifies FIFO waiters on failure, and tracks the server process for reliable cleanup. HTTP 416 trace matching now accounts for both older and current libcurl output formats.

**Why it matters:** The race conditions could corrupt temporary pack files or cause resume requests to fail at EOF, particularly in environments with concurrent Git processes fetching the same packfile into a single object database. The fix is narrowly scoped to the packfile URI and dumb HTTP download paths, with no impact on other HTTP fetch behaviors or on-disk formats.

**Next steps:** The series is rebased onto current `master` and ready for review. If the test suite passes cleanly, it is likely to progress to `next` soon.

---

### **`git rebase --update-refs` symref bugfix nears completion**
**Headline:** Son Luong Ngoc's v3 series fixing `git rebase --update-refs` edge cases with symbolic references is under active review, with Phillip Wood's latest feedback refining the implementation. The series addresses scenarios where symbolic references to branches or non-branch refs cause rebase failures during ref updates.

**What changed:** Phillip's review identified a potential bug in the HEAD comparison logic (using `decoration->name` instead of `resolved_ref`) and reiterated requests for an explanatory comment and minor test organization tweaks. The series remains functionally complete, with no open technical questions.

**Why it matters:** The bug affects branch rename migration scenarios and cross-worktree workflows where symrefs coexist. The fix ensures that `git rebase --update-refs` correctly handles symref decorations, preventing rebase failures even when the primary branch rebases successfully.

**Next steps:** Son is expected to address the refinements in a v4 revision.

---

## In brief

- **`git show-branch` slab conversion:** Gatla Vishweshwar Reddy's refactoring of `git show-branch` to use a commit-slab instead of `object.flags` is technically ready for integration. Junio has not yet commented on the v10 patch, but Gatla's self-review identified minor mechanical issues (indentation, variable scoping) and a test-coverage gap for `name_slab` initialization. A minimal smoke test may be added in v11.

- **`git diff --relative --cached` segfault fixed:** Jeff King's bugfix for a NULL pointer dereference in `git diff --relative --cached` when operating on unmerged paths outside the current directory has been **reviewed and accepted by Junio C Hamano**. The fix adds a NULL check before `fill_filespec()` and includes a regression test.

- **`git config --url` gains scp-style URL support:** Fabian Pottbäcker's three-patch series adds scp-style SSH URL support (e.g., `host:path`) to `git config --url`, bringing it into consistency with other Git commands. Ben Knoble identified a minor comment nit in the first patch, but the series is otherwise uncontroversial.

- **`git repo info` path-discovery design clarified:** Junio C Hamano clarified that the `path.index` keys in K Jayatheerth's `git repo info` series should return the path held by Git's internal `repo->index_file` member, regardless of whether the repository is bare or the file exists. This mirrors the behavior of `git rev-parse --git-path index` and aligns with the precedent set by `path.grafts` in the same series.

- **`git rebase -i` fixup/squash edge cases:** Phillip Wood's two-patch series fixing edge cases in `git rebase -i` (incorrect commit count in template messages and skipped `fixup -c` commands) received substantive reviews from Junio. The first patch's reading logic must handle trailing newlines in legacy command-list files, and the second patch's string matching could be tightened to require a space after the verb.

- **macOS fsmonitor performance regression fixed:** Tamir Duberstein's bugfix for a macOS-specific performance regression in Git's fsmonitor daemon (introduced by 56cef9cb1a) has been reviewed by Koji Nakamaru, who suggested additional benchmark data in the commit message. The patch adds an asynchronous flush of pending FSEvents notifications to prevent timeouts during cookie waits.

- **Submodule remote matching with `url.<base>.insteadOf`:** Éric NICOLAS's bugfix for `git submodule update --remote` failing to match submodule remotes when URL aliases are in use has been **merged to `master`**. The fix ensures that `.gitmodules` URLs are resolved via `alias_url()` before comparison.

- **macOS regex memory leak workaround:** Chungmin Lee's patch adding a Darwin-specific workaround for a memory leak in Git's macOS builds when the system regex engine (TRE) processes invalid UTF-8 sequences received structural feedback from Junio. The patch splits input buffers at invalid multibyte boundaries to avoid the leaking code path, but Junio raised questions about long-term maintainability and the organization of platform-specific overrides.

- **`git clone --revision` segfault fixed:** Adrian Friedli's bugfix for a segmentation fault in `git clone --revision=<ref>` when the server advertises more refs than requested under protocol v0 has been **merged to `master`**. The fix adds a NULL check in `update_remote_refs()` and includes a regression test.

- **Packfile duplicate object handling hardened:** Taylor Blau's five-patch series hardening Git's packfile subsystem against duplicate object entries received an implicit Ack from Junio C Hamano. The series ensures early detection and rejection of invalid packs containing duplicates, addressing edge cases in reverse indexes, delta resolution, MIDX verification, and bitmap generation.

---

## On the radar

- **GSoC 2026 – Improving disk space recovery for partial clones:** Siddharth Shrimali's progress report for weeks 7–8 continues the design and prototyping work on `git maintenance run --task=gc-promisor`. The work remains in the formative phase, with performance testing on large promisor packs underway and edge cases (shallow clones, grafted history) still under evaluation.

- **`git maintenance` gc-promisor task:** The proposed `git maintenance run --task=gc-promisor` subcommand and `maintenance.gcPromisor.enabled` config key remain under design discussion. The grace period (default 30 days) is now configurable via both config (`maintenance.gcPromisor.gracePeriod`) and CLI (`--grace-period=<days>`), and the per-remote `.keep`-file safety mechanism has been implemented.

- **`git rebase --continue --[no-]edit`:** Hugo Santos's patch adding `--[no-]edit` to `git rebase --continue` and a `rebase.noEdit` config knob is under review. The feature allows users to control whether the editor is opened after `rebase --continue` to edit the commit message.

- **`git stash` sparse-index optimization:** Toon Nyman's patch skipping sparse-index expansion for wholly in-cone pathspecs is under review. The patch fixes bounds-safety in a pathspec helper and addresses a performance regression in `git stash`.

- **`git send-pack` no-ref-delta capability:** Taylor Blau's patch teaching `git send-pack` to honor the `no-ref-delta` capability is under review. The feature allows clients to request that the server avoid using REF_DELTA objects in the generated packfile.