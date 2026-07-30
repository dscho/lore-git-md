The day in brief
Wednesday, 2026-07-29 brought **72 emails across 24 threads**, a typical mid-week volume. The standout events: **Harald Nordgren’s `git branch --delete-merged` series hit a correctness wall** after Phillip Wood’s review exposed two blocking issues in the stacked-branch protection and upstream push-check logic. Meanwhile, **Junio ejected `hn/history-squash` from `next`** after Phillip’s critique revealed fundamental reachability flaws, prompting a maintainer policy discussion about review bandwidth. On the feature front, **Pablo Sabater’s GSoC series adding `%(objecttype)` to `git cat-file --batch-command` remote-object-info queries** saw active review, while **Junio’s `git add --resolved` series** sparked a design debate over whether the new option should remain monolithic or split into a more general `--unmerged` selector.

---

### Notable threads

**`git branch --delete-merged` v23 blocked on correctness**
Harald Nordgren’s 23rd iteration of the `--delete-merged` series—implementation-complete and previously marked ready for merging—was derailed by Phillip Wood’s review. The review identified two **blocking technical issues**:
1. **Stacked-branch protection test discrepancy**: The test expects the upstream config of a kept merged branch (`lower`) to remain intact, but the design goal is to **clear stale configs for kept (merged) branches whose own upstream is being deleted**. Harald confirmed the intent is to preserve the upstream chain of surviving *tips*, but the test expectation contradicts this.
2. **`branch_pushes_to_upstream()` correctness**: The current implementation risks **false negatives** (deleting branches that should be preserved, e.g., `main` tracking `origin/main`) or **false positives** (keeping branches that should be deleted). The correct logic should apply the *push* refspec to the local branch, apply the *fetch* refspec in reverse to the result, and compare that to the upstream branch.

Junio’s prior approval is now contingent on resolving these issues. The series remains in `next` but is effectively blocked until Harald addresses the flaws. **Key files**: `builtin/branch.c`, `t/t3200-branch.sh`.

---

**`hn/history-squash` ejected from `next` after reachability flaws exposed**
Phillip Wood’s review of Toon Claes’s `git history squash` subcommand revealed **fundamental correctness issues** in the reachability logic:
- The implementation incorrectly assumes `UNINTERESTING` commits imply a `BOTTOM` commit or preclude root commits.
- It fails to handle multi-tip histories (e.g., `^A C D` in `-A-B-C \ D`).
- The `--reedit-message` option name is misleading, as it creates a new commit rather than re-editing an existing one.

Junio immediately ejected the topic from `next`, calling it "a disaster waiting to happen." The episode prompted a **maintainer policy discussion**: Junio proposed a temporary moratorium on accepting new topics into `seen` unless they receive substantive review, aiming to prevent premature promotion to `next`. Phillip is now developing fixups, but the topic will need to re-prove its correctness before reconsideration. **Key files**: `builtin/history.c`, `revision.c`.

---

**`git add --resolved` design debate: monolithic vs. modular**
Junio’s v2 series introducing `git add --resolved`—which stages only paths whose conflict markers have been removed—sparked a **conceptual critique** from Michael Montalbo. Montalbo argued that the option conflates two distinct operations: selecting unmerged paths (a structural property of the index) and enforcing a conflict-marker safety policy (a content-based check). He proposed an alternative design: a more general `--unmerged` selector with explicit policy flags like `--allow-conflict-markers` or `--skip-conflict-markers`. The debate centers on whether the interface should align with the underlying implementation (modular) or prioritize workflow-oriented simplicity (monolithic). **Key files**: `builtin/add.c`, `t/t2207-add-resolved.sh`.

---

**GSoC: `%(objecttype)` for `git cat-file --batch-command` remote-object-info**
Pablo Sabater’s GSoC series extending `git cat-file --batch-command` to support the `%(objecttype)` placeholder in remote-object-info queries saw **active review** from Chandra Pratap and Karthik Nayak. The series is now complete at v1, with all feedback addressed. Key changes:
- Patch reordering for narrative clarity: the dynamic request logic (patch 3) will move to the front of the series in v2.
- Optional robustness test: Chandra suggested adding a test for malformed server responses (e.g., truncated packets), though this may be deferred to a follow-up.
- Documentation and test updates: Karthik’s feedback on commit messages and test design was incorporated.

The series is **ready for maintainer consideration** after v2 is posted. **Key files**: `protocol-caps.c`, `fetch-object-info.c`, `t/t1017-cat-file-remote-object-info.sh`.

---

**`git rebase --update-refs` symref bugfix: design resolved**
Son Luong Ngoc’s bugfix series for `git rebase --update-refs` behavior with branch symrefs reached **design consensus** after Phillip Wood provided concrete evidence that the edge case (HEAD symrefs pointing outside `refs/heads/`) is actively used by tools like TopGit. Junio conceded the argument, and the robustness approach in the current patch (using `resolved_ref` for HEAD comparison) will be retained in v4. The latent `head_ref` buffer clobbering bug remains unaddressed but is now a candidate for a follow-up patch. **Key files**: `sequencer.c`, `t/t3404-rebase-interactive.sh`.

---

### In brief

**Sparse-index ITA crash fix** – Derrick Stolee’s patch fixing a segfault when collapsing a full index to a sparse index with an intent-to-add (ITA) entry outside the cone was **endorsed by Junio** and marked for `next`. The fix checks if `entry_count` is negative before attempting to collapse a cache-tree node. **Files**: `sparse-index.c`, `t/t1092-sparse-checkout-compatibility.sh`.

**`git add -e` refactoring** – Gatla Vishwahwar Reddy’s patch refactoring `git add -e` to use Git’s internal `apply_all_patches()` API was **approved by Junio** and marked for `next`. The v4 update resolved the subdirectory correctness issue by passing `NULL` as the prefix to `init_apply_state()`. **Files**: `builtin/add.c`, `t/t3702-add-edit.sh`.

**Packfile URI race-condition fix** – Ted Nyman’s v6 series fixing race conditions in Git’s HTTP transport when multiple processes concurrently fetch the same packfile URI was **marked "ready for merge"** by Junio. The series is now complete, with all advisory feedback from Peff addressed. **Files**: `http.c`, `t/t5550-http-fetch-dumb.sh`.

**Trace2 recursion crash** – Junio followed up on the trace2 recursion crash thread, asking whether progress had been made on implementing Taylor Blau’s `xsnprintf()`/`libintl` boundary fix in a MinGW-specific file. No new patches were posted. **Files**: `trace2/tr2_tbuf.c`, `compat/mingw/`.

**`git repo info` path keys** – K Jayatheerth’s series adding seven new `path.*` keys to `git repo info` saw **design discussion** about `/dev/null` handling for `path.hooks` and the long-term viability of exposing the object database directory in light of pluggable ODB backends. The author agreed to document the current behavior (passing `/dev/null` through unchanged) and add a test to lock it in. **Files**: `builtin/repo.c`, `t/t1900-repo-info.sh`.

**`git branch -d` bisect error message** – René Scharfe’s patch clarifying the error message when deleting a branch used by `git bisect` was **endorsed by Junio and Phillip Wood** and is now cooking in `next`. The patch replaces the generic "used by worktree" message with a specific "for bisect" message. **Files**: `branch.c`, `t/t3200-branch.sh`.

**Documentation synopsis-style conversion** – Jean-Noël Avila’s v2 series converting the man pages for `git imap-send`, `git format-patch`, `git send-email`, and `git request-pull` to the new AsciiDoc synopsis style was **approved by Junio** and marked for `next`. **Files**: `Documentation/git-*.adoc`.

**macOS regex leak workaround** – Chungmin Lee’s patch adding a Darwin-specific workaround for a memory leak in Git’s macOS builds when the system regex engine (TRE) processes invalid UTF-8 sequences was **endorsed by Junio**. The fix splits input buffers at invalid multibyte boundaries and searches each valid segment separately. **Files**: `compat/darwin.h`, `compat/darwin/regexec.c`, `t/t7810-grep.sh`.

**`git merge-base --is-ancestor` tests** – Nikolaus Schuetz’s patch adding test coverage for `git merge-base --is-ancestor` was **reviewed by Phillip Wood and Junio**, who discussed the trade-offs of reusing the existing test repository setup versus creating a dedicated one. The patch remains queued in `seen`. **Files**: `t/t6010-merge-base.sh`.

**`commit_body()` test helper** – Shlok Kulshreshtha’s v2 series introducing a reusable `commit_body()` helper in the Git test suite was **acknowledged by Junio** and marked for `next`. The series addresses a long-standing exit-code masking issue in the test suite. **Files**: `t/test-lib-functions.sh`, `t/README`.

**`git cat-file --batch-command` assertion fix** – Jeff King’s patch fixing an assertion failure in `git cat-file --batch-command` when the output format string does not include `%(objecttype)` was **acknowledged by Alan Stokes** and remains queued for `next`. **Files**: `builtin/cat-file.c`, `t/t1006-cat-file.sh`.

**`WITH_BREAKING_CHANGES` test failures** – The discussion about test failures with `WITH_BREAKING_CHANGES=1` was **fully resolved**. Jeff King’s two-patch series fixed the `t0014-alias.sh` failures by dynamically generating the list of deprecated commands, and brian m. carlson’s follow-up patch addressed the root cause of stale build artifacts by ensuring `WITH_BREAKING_CHANGES=1` is respected during initial build target generation. **Files**: `t/t0014-alias.sh`, `Makefile`.

**`pack.allowDuplicateObjects` RFC** – The RFC patch introducing `pack.allowDuplicateObjects` and `--[no-]allow-duplicate-objects` for `git index-pack` saw **philosophical pushback** from Junio, who argued that Git should not tolerate "broken" packfiles. Taylor Blau countered that packs with duplicate objects are not inherently broken—only unsupported by current client-side logic—and cited existing Git behavior (non-strict `index-pack`, shallow clones) as precedent. The discussion remains at an impasse, with Junio suggesting that duplicates should be deduplicated *during* `index-pack` rather than stored on disk. **Files**: `builtin/index-pack.c`, `t/t5308-pack-detect-duplicates.sh`.

**`fetch.bundleURI` performance** – The inquiry into `fetch.bundleURI` slowness when cloning the Linux kernel repository was **resolved**. Enabling threaded `index-pack` via `pack.threads=0` halved the clone time (from 4–5 minutes to ~2 minutes). The discussion shifted to exploring further optimizations, such as incremental processing or bundle generation parameters. **Files**: `index-pack.c`.

**`git maintenance` regression fix** – David Lin’s patch fixing a regression in `git maintenance`’s handling of the `"none"` strategy was posted. The patch adds a case-insensitive check for `"none"` in the parser and updates the test suite. **Files**: `builtin/gc.c`, `t/t7900-maintenance.sh`.

**Hex object IDs to lowercase only (RFC)** – brian m. carlson’s RFC series proposing a Git 3.0 breaking change to restrict hex object IDs to lowercase only was posted. The series introduces infrastructure to enforce lowercase-only parsing and activates it when `WITH_BREAKING_CHANGES` is enabled. The discussion remains in the RFC stage, with no maintainer decision yet. **Files**: `hex-ll.c`, `hex-ll.h`, `object-name.c`, `Documentation/BreakingChanges.adoc`.

---

### On the radar
- **`hn/branch-delete-merged`**: Blocked on Harald Nordgren’s response to Phillip Wood’s review. The upstream push-check logic and merged-hierarchy behavior need resolution.
- **`hn/history-squash`**: Phillip Wood is developing fixups for the reachability logic flaws. The topic will need to re-prove its correctness before reconsideration.
- **`ps/cat-file-remote-object-info-type`**: Pablo Sabater’s GSoC series is ready for v2, which will reorder patches for narrative clarity.
- **`jc/add-resolved`**: Junio’s series awaits a response to Michael Montalbo’s design critique. The debate over `--resolved` vs. `--unmerged` remains unresolved.