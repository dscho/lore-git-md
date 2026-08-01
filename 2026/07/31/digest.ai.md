**The day in brief.**
Friday, July 31, 2026 brought a moderate volume of traffic (44 emails in 16 threads) with a mix of routine progress and lingering design debates. **Two items stand out:** Junio ejected `hn/history-squash` from `next` after Phillip Wood’s review exposed fundamental correctness flaws, sparking a broader discussion about review depth and integration pipeline quality; and Pablo Sabater’s GSoC series extending `git cat-file --batch-command` to support `%(objecttype)` reached v2, now complete and ready for maintainer consideration. The day also saw a steady stream of bug reports, documentation cleanups, and CI tweaks, but no major milestones or contentious flamewars.

---

## Notable threads

### `hn/history-squash` ejected from `next` after correctness review
The `hn/history-squash` topic, which adds a `git history squash` command for multi-commit squashing, was ejected from `next` after Phillip Wood’s review identified critical flaws in its reachability logic and user experience. The implementation incorrectly assumed `UNINTERESTING` commits imply a `BOTTOM` commit or preclude root commits, violating Git’s revision traversal semantics. Junio’s ejection email framed the incident as a symptom of broader review quality issues, proposing a temporary moratorium on accepting new topics into `seen` unless they receive substantive review from the community.

The discussion has since expanded into a procedural debate. Matt Hunter reflected on his earlier surface-level review ("lgtm"), acknowledging it was based on limited scrutiny and raising questions about how reviewers should signal engagement depth. Junio’s response subtly validated Matt’s concern while reinforcing the expectation that reviewers demonstrate deeper engagement to catch correctness flaws before topics reach `next`. Phillip Wood is actively developing fixups to address the reachability logic flaws, but the episode has already prompted a re-evaluation of the project’s integration pipeline. The thread remains open, with no concrete policy changes yet, but the incident may influence future discussions about reviewer incentives, CI integration, or stricter review requirements.

---

### GSoC: `git cat-file --batch-command` gains `%(objecttype)` support
Pablo Sabater’s Google Summer of Code series extending `git cat-file --batch-command` to support the `%(objecttype)` placeholder reached v2, now complete and ready for maintainer consideration. The series builds on Eric Sunshine’s recently merged `remote-object-info` functionality, adding end-to-end support for object type metadata—the last piece of metadata guaranteed to match between client and server after a fetch. The implementation is narrowly scoped to object type, with other metadata (disk size, delta base, object mode) explicitly ruled out as unreliable or context-dependent.

The v2 update addresses all review feedback from v1, including patch reordering for narrative clarity, a test portability fix (replacing `wc -c <file | xargs` with Git’s `test_file_size` helper), and a unified default format string for both local and remote queries. Junio’s review of the first patch was confirmatory, tracing the data flow to validate the safety of the dynamic request logic. The series is now feature-complete, with expanded test coverage in `t1017` and `t5701`, and no open questions or loose ends. The only remaining step is Junio’s merge decision, which is likely to come in the next "What's cooking" cycle.

---

### `git repack --drop-filtered` for partial clones: design questions persist
Siddharth Shrimali’s RFC v2 series introducing `git repack --drop-filtered` for partial clones remains under review, with Junio raising three substantive technical questions about the design. The series extends `git repack` with a `--drop-filtered` option that safely reclaims disk space by removing locally cached promisor blobs exceeding a user-specified size threshold (e.g., `--filter=blob:limit=10M`). Junio’s feedback probes the reliability of `ODB_FOR_EACH_OBJECT_PROMISOR_ONLY`, the repack machinery’s behavior (local-only construction, no lazy fetches), and the necessity of the safety guards (merge/rebase/cherry-pick checks and index validation).

The safety guards, in particular, have become a focal point. Junio questions whether they are redundant given the promisor infrastructure’s lazy-fetch guarantees, presenting scenarios where lazy-fetching could transparently restore dropped objects. Siddharth has not yet responded to this critique, leaving the design direction unresolved. The series is technically complete—addressing all blocking issues from v1, including a critical validation flaw in the `--drop-filtered` vs `--write-bitmap-index` logic—but the open questions may require a v3 or further discussion before advancing to `next`.

---

### `git add --resolved`: workflow-oriented vs. flexible design
Junio posted v3 of his `git add --resolved` series, a workflow-oriented feature that stages only paths whose conflict markers have been removed. The series consolidates duplicate conflict-marker detection logic, adds helpers for index removal with flags, and includes a performance optimization for binary files. The implementation is technically complete, addressing all prior review feedback, but the user-facing interface remains contested.

Michael Montalbo proposed an alternative design: a more general `--unmerged` selector with explicit policy flags (e.g., `--allow-conflict-markers`), arguing this would align better with the underlying implementation and offer greater flexibility. Junio defends `--resolved` as a monolithic, workflow-oriented operation, citing consistency with other Git commands (e.g., `git am --resolved`). The debate highlights a tension between scriptability (Montalbo’s `--no-edit` proposal) and user intent (Junio’s "editor always opens" default). No consensus has emerged, but the series is ready for review on its technical merits, with the interface design as the primary open question.

---

### Git 3.0: lowercase-only hex object IDs debated
brian m. carlson’s RFC series proposing a Git 3.0 breaking change to restrict hex object IDs to lowercase only continues to generate discussion, with Junio challenging the fundamental premise. The series aims to enforce lowercase-only parsing to address real-world security vulnerabilities in external tools that assume hex uniqueness, but Junio has invoked the Robustness Principle to argue that Git should remain liberal in what it accepts, even in a breaking-change release.

Junio’s latest email expands on this objection with a concrete example: Git’s dumb HTTP transport encountering a loose object directory named in uppercase (e.g., `objects/AB/`). He suggests that a robust implementation would accept such directories, normalize them to lowercase internally, and avoid storing duplicates—rather than rejecting them outright. The discussion remains focused on the justification for the change, with no maintainer decision or merge status yet. The first five patches (infrastructure-only) are uncontroversial, but the sixth patch (enforcement) hinges on resolving the broader design question.

---

## In brief

**`git config --global` reads only one file despite docs** — Nils Fahldieck reports that `--global` reads only `~/.gitconfig` (prioritizing it even if empty) while the documentation claims it reads both `~/.gitconfig` and the XDG config file. Ben Knoble and Junio agree the docs are wrong, but no consensus has emerged on whether to fix the code (read both files) or the docs (document single-file behavior). Nils has signaled willingness to contribute a patch but is uncertain about the documentation process.

**`git stash push` with paths stashes unrelated files** — Yuri reports that `git stash push` with explicit paths unexpectedly stashes changes from an unrelated directory, including an untracked file not listed on the command line. The issue is reproducible in the FreeBSD ports tree, suggesting a bug in `git stash`’s path-filtering logic or untracked-file handling. No patch or fix proposed yet.

**`git rebase` regression with commit graphs and submodules** — Florian Schmidt reports a regression in `git rebase` when commit graphs are enabled and the rebase involves submodule pointer changes. The error (`invalid commit position. commit-graph is likely corrupt`) occurs because `lookup_commit_reference_gently()` now consults the commit graph but appears to resolve object references across repository boundaries. The regression was introduced by commit `bb5da75d61` ("commit: use commit graph in `lookup_commit_reference_gently()`"). Workaround: disable commit graphs (`core.commitGraph=false`).

**`git-refs(1)` man page improvements** — Kristoffer Haugsbakk’s two-patch series reorganizing a ref migration warning into an admonition block and adding a cross-reference to `git-maintenance(1)` was approved by Junio and queued for integration. The changes are purely presentational, elevating a critical warning about concurrent writes during ref migration and improving navigation.

**CI: cancel stale workflow runs for pull requests** — Harald Nordgren’s patch modifies GitHub Actions workflow configuration to cancel stale in-progress runs for pull requests, reducing wasted CI runner capacity. The change is self-contained and unlikely to be controversial.

**Documentation: `git replay` config standardization** — Kristoffer Haugsbakk’s `kh/doc-replay-config` series, which synchronizes `replay.refAction` documentation between `git-config(1)` and `git-replay(1)`, was marked for `next` after Toon Claes’s final visual verification on git-scm.com. The series is now fully reviewed and queued for graduation to `master`.

**Sparse-index segfault fix** — Derrick Stolee’s patch fixing a segfault in the sparse-index subsystem when collapsing a full index to a sparse index in the presence of an intent-to-add (ITA) entry outside the sparse-checkout cone was confirmed for `next`. The fix adds a guard clause in `convert_to_sparse_rec()` to skip cache-tree nodes with negative `entry_count`, preventing an out-of-bounds access.

**Trace2 recursion crash fix** — Derrick Stolee’s investigation into a "recursion detected in die handler" crash in the trace2 subsystem has identified memory allocation failures as the likely root cause. The fix will replace `xsnprintf()` and `xstrdup()` in trace2 to avoid `die()` calls during `atexit` events, and implement Taylor Blau’s MinGW-specific fix for `xsnprintf()`/`libintl` redirection. No revised patches posted yet.

**Feature request: suppress trailing blank-line warnings** — Thomas Nemeth requested a configuration option to suppress warnings about trailing blank lines at the end of files, motivated by Vim workflow ergonomics. Junio clarified that Git currently lacks a dedicated mechanism to suppress these warnings without also disabling other whitespace checks (e.g., via `core.whitespace=-blank-at-eof`). The thread remains dormant, with no maintainer intention to implement or merge expressed.

---

## On the radar
- **`hn/branch-delete-merged`** remains in `next` but faces unresolved design questions about upstream push-check logic and merged-hierarchy behavior. Junio has deferred to community consensus, but no fixups or further discussion have emerged.
- **`hn/history-squash`** is blocked pending Phillip Wood’s fixups for the reachability logic flaws. The broader procedural discussion about review quality and integration pipeline policy remains open-ended.
- **Rustification effort** continues to generate debate, with Randall S. Becker’s concerns about platform support (NonStop) still unaddressed. No new patches or design proposals have surfaced.