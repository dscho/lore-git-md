# The Git Mailing List Daily Digest for 2026/07/30

## The day in brief

A busy Thursday on the Git mailing list, with **73 emails across 20 threads** covering everything from documentation polish to breaking changes for Git 3.0. The most consequential developments: **Harald Nordgren’s `git branch --delete-merged` series reached v24**, addressing the last blocking issues and positioning itself for merging; **Junio ejected `hn/history-squash` from `next`** after Phillip Wood’s review exposed fundamental correctness flaws; and **brian m. carlson’s RFC for lowercase-only hex object IDs** sparked a debate about whether the security benefits justify a breaking change. Documentation efforts continued apace, with Kristoffer Haugsbakk reviving two stalled series and submitting new patches for `git replay` and `git interpret-trailers`.

---

## Notable threads

### **`git branch --delete-merged` reaches v24, ready for merging**
**Topic:** `hn/branch-delete-merged` (Harald Nordgren)
**Status:** Implementation-complete, ready for `next`

Harald Nordgren’s long-running series adding `--delete-merged` to `git branch` reached its 24th iteration, addressing the last two blocking issues identified by Phillip Wood: a **flaw in the push-detection logic** and a **test discrepancy in the stacked-branch protection design**. The v24 changes are surgical:
- The `branch_pushes_to_upstream()` function now correctly applies the *push* refspec to the local branch, then the *fetch* refspec in reverse, ensuring accurate detection regardless of `remote.pushDefault` configuration.
- The stacked-branch protection logic was simplified to clear upstream configs for kept (merged) branches whose own upstream is being deleted, aligning with the design goal and resolving the test mismatch.

The series is now **technically complete**, with all planned features (per-branch opt-out, `--dry-run` support, order-independent stacked-branch protection) and safety mechanisms in place. Junio C Hamano has confirmed the series is ready for merging pending final review of the v24 fixes, and Phillip Wood’s blocking concerns have been resolved. The only agreed follow-up is extending stacked-branch protection to `git branch -d` for consistency, which will be implemented after the series lands.

**Why it matters:** This feature addresses a real workflow pain point for users managing topic branches, offering safe, automated cleanup of merged branches with precise control over which upstreams to consider. The series has been thoroughly tested (146 new test lines) and aligns with Git’s API consistency expectations.

---

### **`hn/history-squash` ejected from `next` after correctness flaws exposed**
**Topic:** `hn/history-squash` (Harald Nordgren)
**Status:** Blocked pending fixups

Junio C Hamano ejected Harald Nordgren’s `git history squash` subcommand from `next` after Phillip Wood’s review exposed **fundamental flaws in the reachability logic** and **UX inconsistencies**. The core issues:
- The implementation fails to handle `UNINTERESTING` commits (e.g., `git history squash ^A B` where `A` is `UNINTERESTING`), root commits, and multi-tip histories (e.g., `git history squash ^A C D` in `-A-B-C \ D`).
- The command refuses to squash commits pointed to by tags or remote-tracking refs, unlike other `git history` commands, creating an inconsistency.
- The `--reedit-message` option and message template reuse (from `git rebase`) are under scrutiny for suitability in multi-commit squashing.

Phillip Wood is actively developing fixups to address the reachability logic flaws, but the series remains blocked until these are submitted. Harald has proposed removing `--reedit-message` and making re-editing the default behavior, but this does not address Phillip’s broader concerns about preserving `amend!`/`squash!` messages or the inconsistency with other `git history` commands.

**Why it matters:** The ejection highlights the importance of thorough review before topics graduate to `next`. Junio’s response also signals heightened scrutiny for future merges, with a proposed moratorium on accepting new topics into `seen` unless they receive substantive review.

---

### **RFC: Restrict hex object IDs to lowercase in Git 3.0**
**Topic:** `bc/object-id-lowercase` (brian m. carlson)
**Status:** Design debate ongoing

brian m. carlson’s RFC proposing to restrict hex object IDs to lowercase only in Git 3.0 sparked a debate about whether the security benefits justify the breaking change. The series introduces infrastructure to enforce lowercase parsing (gated behind `WITH_BREAKING_CHANGES`) and documents the change in `BreakingChanges.adoc`. Junio C Hamano initially pushed back on the motivation, calling it insufficient, but brian followed up with **concrete evidence of real-world security vulnerabilities** caused by Git’s current case-insensitive parsing, citing multiple instances where external tools failed to properly sanitize or filter object IDs due to hex uniqueness assumptions.

The discussion now centers on the trade-off between disruption and security. brian argues that the change aligns with ecosystem expectations (e.g., Gitolite and GitHub-hosted tools reject or mishandle uppercase hex) and Git’s own documentation (hook examples use lowercase-only patterns). Junio has not yet responded to the new evidence, leaving the series’ future uncertain.

**Why it matters:** This is a rare breaking change proposal for Git 3.0, touching core object ID parsing. The outcome will set a precedent for how the project balances security, consistency, and backward compatibility in major releases.

---

### **`git repack --drop-filtered` for partial clones (RFC v2)**
**Topic:** `ss/repack-drop-filtered` (Siddharth Shrimali)
**Status:** Under substantive review

Siddharth Shrimali’s RFC v2 for `git repack --drop-filtered` is now complete and ready for review. The series adds a `--drop-filtered` option to `git repack` that safely reclaims disk space in partial clones by removing locally cached promisor blobs exceeding a user-specified size threshold (e.g., `--filter=blob:limit=10M`). Key features:
- **Safety guards**: Prevents drops during merges, rebases, or cherry-picks, and refuses to drop blobs referenced by the current index.
- **Core logic**: Uses `odb_for_each_object()` with `ODB_FOR_EACH_OBJECT_PROMISOR_ONLY` to enumerate promisor blobs and `repack_promisor_objects()` to rebuild packs while excluding dropped blobs.
- **Documentation**: Adds user-facing descriptions of `--drop-filtered` and `--dry-run`, including constraints (e.g., `blob:limit` only, `-a` required, promisor remote required) and incompatibilities (e.g., `--filter-to`, bitmaps).

The series is well-structured, with tests distributed across the relevant commits and safety mechanisms implemented to prevent data loss. The deferred work (drop log, remote verification) is explicitly scoped and does not block the core functionality. Reviewers with expertise in partial clones or repack optimizations (e.g., Patrick Steinhardt, Christian Couder) are likely to weigh in.

**Why it matters:** This feature addresses a real pain point for partial clone users, enabling safe disk space reclamation without risking data loss. The implementation leverages existing repack crash-safety guarantees and is motivated by a clear use case.

---

### **Documentation overhauls: `git interpret-trailers` and `git replay`**
**Topics:** `kh/interpret-trailers-doc` (v4), `kh/doc-replay-config` (v1) (Kristoffer Haugsbakk)
**Status:** Revived, under review

Kristoffer Haugsbakk revived two previously stalled documentation series in response to Junio’s "What’s cooking" report, where both were marked as "Will discard" pending updates:
1. **`git interpret-trailers` (v4)**: Overhauls the man page to clarify terminology (e.g., "trailer block", "metadata"), explain the strict key format (alphanumeric + hyphens), and document how comment lines are handled. The series removes outdated RFC 822 references and adds concrete failure-case examples.
2. **`git replay` (v1)**: Improves the clarity and structure of the configuration documentation by reorganizing the description list, moving the "default" value to the right-hand side, and adding cross-references to the config section.

Both series are now back in play for review, having addressed prior feedback. The `interpret-trailers` overhaul is particularly notable for its attention to detail, including a debate about singular vs. plural terminology ("metadata" vs. "trailers metadata") and the resolution of ambiguity around blank line insertion behavior.

**Why it matters:** These efforts align with the broader project goal of modernizing Git’s documentation using consistent AsciiDoc markup. The `interpret-trailers` series, in particular, demonstrates how even small phrasing choices are debated to ensure clarity.

---

## In brief

- **`git mv` symlink-handling bugfix merged to `next`** (Lucas Zamboni Orioli): The two-patch series improves error messages and adds an early check for missing or problematic destination directories, including rejecting any tracked symlink in the destination path to prevent index corruption. Junio explicitly approved the approach, and the series is now in `next`.
- **`git rebase --update-refs` symref bugfix (v4 anticipated)** (Son Luong Ngoc): Phillip Wood clarified a detail about TopGit’s ref naming conventions, underscoring why Git cannot tighten refname restrictions. The series remains under review, with v4 expected to address the last minor refinements.
- **`git add --resolved` design debate** (Junio C Hamano): Junio defended the `--resolved` option as a monolithic, workflow-oriented operation, while Michael Montalbo proposed splitting it into a more general `--unmerged` selector with explicit policy flags. The discussion remains at a conceptual impasse.
- **`git maintenance` regression fix** (David Lin): Junio reviewed the patch, requesting an in-body `From:` line to align author and sign-off addresses. The fix adds case-insensitive `"none"` support to `parse_maintenance_strategy()` and is ready for merging.
- **`git stash drop`/`pop` safety series retracted** (Junio C Hamano): The two-patch series was withdrawn after Junio discovered the reflog expiration machinery already mitigates the original concern (silent removal of multiple stash entries).
- **`--date=elapsed` and `--date=elapsed-pad`** (Israel Roldan): A new feature adding precise, script-friendly elapsed-time formatting (e.g., "1y 11M 5d 13h 5m 13s") to Git, modeled after `uptime(1)`. The patch is well-structured and passes all tests, with open questions about placeholder letter choice and format naming.
- **`git config --global` documentation discrepancy** (Nils Fahldieck): Junio agreed the documentation is incorrect but did not endorse a specific fix. The thread remains unresolved on whether to align code with docs (read both files) or docs with code (document single-file behavior).

---

## On the radar

- **ODB abstraction series** (`ps/odb-pluggable-housekeeping`): Patrick Steinhardt’s series to make on-disk ODB structures pluggable remains under review, with open questions about downcast safety and multi-backend consistency.
- **`git history squash` fixups**: Phillip Wood is developing fixups to address the reachability logic flaws in `hn/history-squash`. The series’ future depends on these being submitted and reviewed.
- **Lowercase-only hex object IDs**: brian m. carlson’s RFC awaits Junio’s response to the new evidence of real-world security vulnerabilities. The outcome will determine whether this breaking change is adopted for Git 3.0.