## The day in brief

Saturday, July 25, 2026, was a **busy but focused day** on the Git mailing list, with **47 emails across 14 threads**. The standout event was the **final approval and readiness for merging** of Harald Nordgren’s `git branch --delete-merged` series after 23 iterations, but the day also saw **critical bugfixes**, **substantive reviews**, and **GSoC progress** on remote object metadata. The most urgent items: **a memory-safety bug in `git worktree add` that can cause data loss**, and **the packfile URI race-condition series still blocked by test flakiness**.

---

## Notable threads

### **`git branch --delete-merged` lands after 23 iterations**
**Headline:** Harald Nordgren’s long-running series adding `--delete-merged` to `git branch` is **now ready for merging** after Junio C Hamano’s final approval.

The series introduces a **safe, automated local-branch cleanup command** with comprehensive safety controls: **order-independent stacked-branch protection**, **per-branch opt-out** (`branch.<name>.deleteMerged=false`), **`--dry-run` preview**, and **repeatable upstream selectors**. The v23 update addressed the last mechanical issues (type mismatch and style nit in the `--forked` filter), and Junio confirmed there are **no outstanding issues**. The series will appear in an upcoming "What's cooking" report and is expected to graduate to `next` soon.

The only agreed follow-up is extending stacked-branch protection to `git branch -d` for consistency, which Harald will implement post-merge. The series is **well-tested**, with 204 lines of new tests covering all edge cases (self-referential upstreams, missing upstreams, push divergence, and dry-run behavior).

---

### **Packfile URI race conditions: test flakiness blocks progress**
**Headline:** Ted Nyman’s v4 series fixing packfile URI download race conditions remains **blocked by test flakiness**, despite the core logic being sound.

Jeff King (Peff) identified that the intermittent `--stress` failure in `t5550-http-fetch-dumb.sh` (test 34) is a **false positive** caused by a race in the test’s `test -s` synchronization check. The deterministic staging path design ensures overlapping downloads write identical bytes, making the order of writes irrelevant. Peff and Junio agree the fix is to **drop the `test -s` check and the trace greps for HTTP 200/206 responses**, as the FIFO-based synchronization is sufficient. The series is otherwise **technically complete**, with production code unchanged since v3, and the fix is purely about test robustness. Ted will reroll with the proposed test adjustment.

---

### **Memory-safety bug in `git worktree add` can cause data loss**
**Headline:** Matthias Aßhauer posted a **critical bugfix** for `git worktree add` that prevents an out-of-bounds read leading to **silent data corruption** (recursive deletion of the working directory, including `.git`).

The bug surfaces when an empty string or a string of only directory separators (e.g., `""` or `"//"`) is passed as the worktree path, likely via an unset environment variable (e.g., `git worktree add "$UNSET_VAR"`). Junio’s review identified a **logical flaw** in the proposed fix: the patch could still underflow the pointer when the input is a string like `"///"`, potentially causing `strbuf_add()` to allocate an enormous buffer. He suggested a safer approach using an integer index loop to eliminate underflow risk. The series is **under active review**, and a v2 is expected soon.

---

### **`git rebase --update-refs` symref bugfix series advances**
**Headline:** Son Luong Ngoc’s v3 series fixing `git rebase --update-refs` edge cases with symbolic references is **functionally complete**, with only minor test organization tweaks requested.

The series addresses a latent bug where symrefs (e.g., `main` pointing to `master`) cause rebase failures during ref updates. Erik Cervin-Edin’s review confirmed the v3 series resolves a regression in v2 (incorrect current branch skipping) but also surfaced a **pre-existing bug in `master`**: the `head_ref` buffer used for HEAD comparison is clobbered by `branch_checked_out()`, causing branches to incorrectly appear in the rebase todo list. This is not a regression but explains unexpected behavior in multi-worktree setups. Phillip Wood’s latest feedback requests an explanatory comment and minor test organization tweaks. A v4 is expected soon.

---

### **GSoC: `git cat-file --batch-command` gains `%(objecttype)` support**
**Headline:** Pablo Sabater’s GSoC series extends `git cat-file --batch-command` to support the `%(objecttype)` placeholder in remote-object-info queries, completing the metadata puzzle.

The series adds end-to-end support for object type queries, which is the last piece of metadata guaranteed to be identical on client and server after a fetch. The implementation is **mechanical and well-scoped**, following the established pattern from Eric Sunshine’s predecessor series. The patches touch `protocol-caps.c`, `fetch-object-info.c`, `serve.c`, and the test suite, with thorough test coverage and clear documentation updates. The series is **ready for review** and passed CI on GitHub.

---

## In brief

**`git merge-base --is-ancestor` gains test coverage** -- Nikolaus Schuetz added comprehensive tests for the `--is-ancestor` option in `t/t6010-merge-base.sh`, covering exit codes, error conditions, and edge cases (self-ancestry, diverged commits, non-existent commits). The patch is **self-contained and uncontroversial**.

**`git checkout -m` refines autostash conflict handling** -- Harald Nordgren posted a two-patch series that (1) makes the sequencer’s autostash apply logic explicitly report conflicts, and (2) skips the autostash retry loop when no tracked local changes exist. The series also improves the visual separation of autostash conflict advice from the branch-switch confirmation message.

**Memory leaks plugged in `remote.c`** -- Junio C Hamano fixed memory leaks in `rewrites_release()` by properly freeing nested allocations within `struct rewrites`. Jeff King’s review confirmed the v2 approach (explicit string copying) is correct. The patch is **ready for integration**.

**`the_repository` removal: global config migration stalls on design question** -- Tian Yuchen’s series migrating four global configuration variables (`minimum_abbrev`, `default_abbrev`, `pack_size_limit_cfg`, `assume_unchanged`) into `repo_config_values` hit a snag. Junio objected to the unconditional fallback to `the_repository` when `repo != the_repository`, arguing it papers over an API limitation. He proposed `BUG()` as a forcing function to resolve the issue properly. A v2 is expected.

**`git repack --drop-filtered` safety guards confirmed for v2** -- Siddharth Shrimali and Siddharth Asthana agreed that **safety guards** (preventing drops during merges/rebases/cherry-picks and refusing to drop blobs referenced by the current index) are **non-negotiable for v2**. The drop log is deferred to a follow-up, and the CLI design (`--drop-filtered` + `--dry-run`) is finalized.

**`git worktree add` bugfix needs rework** -- Junio’s review of Matthias Aßhauer’s bugfix for `git worktree add` identified a **logical flaw** in the proposed fix: it could still underflow the pointer when the input is a string like `"///"`. He suggested a safer approach using an integer index loop. A v2 is expected.

---

## On the radar

**Packfile URI race conditions** -- Ted Nyman’s series remains **blocked by test flakiness**, but the fix is straightforward (drop the `test -s` check). The core logic is sound, and the series is otherwise ready for `next`.

**`git rebase --update-refs` symref bugfix** -- Son Luong Ngoc’s v3 series is **functionally complete**, with only minor test organization tweaks requested. The latent `head_ref` bug in `master` may become a follow-up topic.

**`the_repository` removal: global config migration** -- Tian Yuchen’s series is **stalled on a design question** about repository-scoped getters. Junio’s objection to the fallback to `the_repository` may require deeper discussion or a v2 with `BUG()` as a forcing function.