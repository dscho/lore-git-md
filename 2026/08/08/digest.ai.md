## The day in brief

**2026/08/08** was a **moderate-volume Saturday** (41 emails across 16 threads) with a **mix of procedural wrap-ups, substantive reviews, and usability discussions**. The standout developments: **Pablo Sabater’s GSoC series for `%(objecttype)` in `git cat-file --batch-command` is now ready for integration** after addressing all review feedback, while **Junio’s "What’s cooking" report** provided the latest snapshot of the integration pipeline, including new topics like `ss/repack-drop-filtered` and `vm/complete-history`. A **usability bug in `git rebase -X`** (silently ignoring invalid options) was reported, and **Elijah Newren’s reviews** on the `git replay --linearize` and `git fast-import` series refined key UX and design details. The day also saw **procedural follow-ups** (e.g., `git repack --drop-filtered` due diligence) and **cosmetic discussions** (gitk color dialog phrasing), but no major controversies or merges.

---

## Notable threads

### `git cat-file --batch-command`: `%(objecttype)` support ready for merge
**Headline:** Pablo Sabater’s **GSoC series extending `git cat-file --batch-command` to support the `%(objecttype)` placeholder** is now **feature-complete and ready for integration** after addressing all review feedback in v6. The series adds end-to-end support for object type metadata in remote-object-info queries, completing the last piece of metadata guaranteed to be identical on client and server after a fetch.

**What’s new:** Junio explicitly endorsed all ten patches as ready to merge, and Chandra Pratap confirmed he has reviewed the entire series and finds it acceptable. The v6 changes were minimal (forward declarations to fix a static-analysis warning and a whitespace fix), and all prior technical concerns—including protocol robustness, design refinements, and test coverage—have been resolved. The series unifies the default format for all `cat-file` batch commands, ensuring consistent output across all transport protocols (git://, file://, http://).

**Key details:**
- **Files touched:** `builtin/cat-file.c`, `fetch-object-info.c`, `protocol-caps.c`, `serve.c`, and related documentation/test files.
- **Behavior:** Client requests and displays object type metadata from the server, matching local queries. The server parses client requests for "type," transmits type data as a string (e.g., `"blob"`), and advertises the "type" capability.
- **Performance:** No regressions; the hybrid design (single array of lightweight structs, caller-controlled field population) avoids sentinel values and boolean flags, streamlining how object metadata is stored and handled.
- **Test coverage:** Expanded in `t5701` (server-side) and `t1017` (client-side), including verification of the `%(objecttype)` placeholder in batch-command output.

**Next steps:** Junio is expected to queue the series to `next` for cooking, after which it will graduate to `master` in a future integration cycle. The potential optimization to make the `remote-object-info` protocol stateful for connection reuse is deferred to a follow-up series.

---

### `git replay --linearize`: UX refinements and single-branch restriction
**Headline:** Elijah Newren’s **review of Toon Claes’s `git replay --linearize` series** (v8) refined the **UX clarity** of the single-branch restriction and proposed extending it to `--revert` mode. The series, which adds a `--linearize` option to flatten merge commits, is now **technically complete** and addresses all prior feedback, including the multi-branch ambiguity that had blocked it in `next`.

**What’s new:** Newren’s review focused on two key UX concerns:
1. **Documentation clarity:** Suggested simplifying the rationale for the single-branch restriction (e.g., "multiple branches are not supported" instead of a technical explanation of emergent reachability issues).
2. **Error message terminology:** Proposed using "branches" instead of "revision ranges" for clarity, given the prior discussion about multi-branch ambiguity.
3. **New technical consideration:** Proposed extending the single-branch restriction to `--revert` mode to prevent concatenating unrelated histories in an unpredictable order (matching `git cherry-pick`'s behavior). This could prompt a small follow-up patch.

**Key details:**
- **Files touched:** `replay.c`, `replay.h`, `builtin/replay.c`, `Documentation/git-replay.adoc`, `t3650-replay-basics.sh`.
- **Behavior:** `--linearize` flattens merge commits into a single linear history, dropping merges and reparenting subsequent commits onto the last non-merge commit. The v8 restriction ensures no emergent reachability issues by disallowing `--linearize` with multiple revision ranges or `--contained`.
- **Test coverage:** Thorough, including cases for replaying to the root commit, merge commit rejection, `--linearize` with `--advance`, `--revert`/`--linearize` compatibility (now restricted to single branches), and verifying the behavioral difference from `git rebase --no-rebase-merges`.

**Next steps:** The series remains in active review, with Junio yet to merge v8 to `next`. The UX refinements are non-blocking but may prompt minor documentation and error-message tweaks before graduation.

---

### `git repack --drop-filtered`: Procedural due diligence
**Headline:** Siddharth Shrimali **acknowledged Junio’s reminder** to perform full due diligence (running the entire test suite and trial merges into `next`/`seen`) before posting the next revision of the `git repack --drop-filtered` series. The series, which enables space reclamation in partial clones by removing locally cached promisor blobs exceeding a size threshold, is **functionally complete at v4** but awaits this procedural step.

**What’s new:** The author confirmed intent to perform the due diligence, which is a routine step for contributors to ensure patches integrate cleanly and do not break existing functionality. No technical changes are expected in the next revision.

**Key details:**
- **Files touched:** `builtin/repack.c`, `repack-filtered.c`, `list-objects-filter.{c,h}`, `repack-promisor.c`, `repack.h`, `Documentation/git-repack.adoc`, `t/t7706-repack-drop-filtered.sh`.
- **Behavior:** `--drop-filtered` + `--filter=<spec>` (currently limited to `blob:limit=<n>`) removes promisor blobs exceeding the threshold; locally created objects are untouched. Safety optimizations (merge/rebase/cherry-pick checks, index validation) prevent drops during ongoing operations or of blobs referenced by the current index.
- **Open question:** Junio’s proposal to replace the current safety guards with a time-based heuristic (avoid culling recently fetched objects) remains unresolved, but the current guards are now viewed as UX optimizations rather than correctness measures.

**Next steps:** The author will perform the due diligence and post v5, after which the series will likely be queued to `next`.

---

### `git fast-import`: Libification and parse-options refactoring
**Headline:** Elijah Newren’s **reviews of Christian Couder’s 12-patch v2 series** refactoring `git fast-import`’s usage string and advancing its libification identified **three minor issues** that remain the only open points. The series is otherwise ready for broader review or integration.

**What’s new:** Newren’s reviews focused on:
1. **Documentation wording** (patch 2/12): Suggested rewording the `PARSE_OPT_LASTARG_DEFAULT` description to clarify its interaction with the command-line parser and its incompatibility with `PARSE_OPT_OPTARG`.
2. **Function signature style** (patch 7/12): Requested reformatting overly long function signatures (e.g., `note_change_n`) to comply with Git’s 80-character line limit.
3. **Behavioral inconsistency** (patch 11/12): Identified a subtle inconsistency between the early scan for `--allow-unsafe-features` and `parse_options()`: the early scan is stricter, only matching the exact spelling and stopping at the first non-option token, while `parse_options()` accepts abbreviations and space-separated values. This can cause the in-stream feature to be rejected even though the command-line option is accepted. Newren proposed documenting this as a known limitation or adjusting the early scan to match `parse_options()`’s flexibility.

**Key details:**
- **Files touched:** `parse-options.h`, `parse-options.c`, `builtin/fast-import.c`, `Documentation/technical/api-parse-options.adoc`, `Documentation/git-fast-import.adoc`, and the test suite (`t0040`, `t0450`, `t9300`).
- **Behavior changes:** Patch 11/12 (the behavior-changing patch) switches the command-line parser from a hand-written loop to `parse_options()`, removing support for five undocumented options (`--alias`, `--get-mark`, `--ls`, `--notes`, `--cat-blob`) on the command line. It also enables abbreviated options (e.g., `--dep=5` for `--depth=5`) and space-separated values (e.g., `--depth 5`).
- **New symbols:** `OPT_HIDDEN_GROUP`, `struct fast_import_state`, and related helper functions.

**Next steps:** The series awaits resolution of the three minor issues, after which it will likely be queued to `next`.

---

### `git last-modified`: Bloom filter optimization edge case
**Headline:** Junio’s **review of Toon Claes’s v2 series** optimizing `git last-modified` with Bloom filters requested a **test to verify the restoration of `bloom_filter_settings` for wildcard pathspecs**. The series achieves a **3.7× speedup** by reusing Bloom filters instead of re-fetching them, and the fix for wildcard pathspecs (patch 6/6) is essential for ensuring the optimization works for all pathspec types.

**What’s new:** Junio asked whether the patch should include a test to verify that the restoration of `bloom_filter_settings` works for wildcard pathspecs (e.g., `*.c`). The concern is valid: while the series includes a test for `--show-trees` and wildcard pathspecs, this specific edge case (the settings being cleared and restored) is not explicitly covered. The failure mode is subtle, and without instrumentation, a test might not show behavioral differences even if the fix is missing.

**Key details:**
- **Files touched:** `revision.c`, `revision.h`, `bloom.c`, `bloom.h`, `builtin/last-modified.c`, `t/t8020-last-modified.sh`.
- **Behavior:** The optimization reuses Bloom filters for pathspec queries, reducing runtime by up to 3.7×. The `--show-trees` correctness bug is fixed by filtering ancestor tree entries not covered by the original pathspec.
- **Test coverage:** Benchmarks included in the cover letter; new test in `t8020-last-modified.sh` verifies Bloom filter behavior with `--show-trees` and wildcard pathspecs.

**Next steps:** The series is otherwise ready for integration, pending this minor test gap.

---

### `git worktree add`: Usability improvement for ambiguous remote branches
**Headline:** Junio’s **review of Yoichi NAKAYAMA’s patch** improving the error message for `git worktree add` when an ambiguous remote branch name is provided pushed for a **more actionable advice message**. The current patch adds a detailed advice message explaining the ambiguity and offering workarounds, but Junio argues it should **list the conflicting remotes** (e.g., "the branch exists on `origin`, `upstream`, and `home`") to address the more common user problem of *discovery* rather than syntax.

**What’s new:** Junio’s latest feedback refines the usability goal:
1. **Discovery vs. syntax:** The current advice assumes the user’s problem is *syntax*—that they don’t know how to specify a fully qualified branch name. But the real problem might be *discovery*: the user may not remember which remotes (`origin`, `upstream`, etc.) even *have* the branch they want. The patch’s advice doesn’t help with that.
2. **Scenario-specific advice:** Junio suggests tailoring the advice to the two distinct scenarios (when `worktree.guessremote` is set and when it isn’t), as the user’s intent (and thus the most helpful suggestion) might differ between them.

**Key details:**
- **Files touched:** `builtin/worktree.c`, `t/t2400-worktree-add.sh`.
- **Behavior:** Advice shown only when ambiguity is detected; the message’s content is the sole point of contention.
- **Test coverage:** Updated existing tests + new test for `--guess-remote` with ambiguous branch.

**Next steps:** A v2 is likely to incorporate Junio’s feedback, focusing on making the advice more actionable by surfacing the specific remotes involved.

---

## In brief

**`git-interpret-trailers` documentation series:** Kristoffer Haugsbakk’s **11-patch documentation series** for `git-interpret-trailers` is now **fully merged to `master`**, with only a trivial v5 planned to address a grammatical nit in the commit message. The series provides comprehensive documentation of valid key formats, parsing rules, and examples, resolving a long-standing ambiguity in the tool’s behavior.

**GSoC `git repo` command:** K Jayatheerth’s **ninth and tenth weekly GSoC updates** for the `git repo` command project continue the established pattern of external progress reporting via blog posts. The project remains in the research/design phase, with no patches, technical specifications, or design documents shared on-list. The mailing list serves as a notification channel rather than a venue for technical discussion.

**Packfile URI race-condition fix:** Junio **confirmed Ted Nyman’s v6 packfile URI race-condition fix series is ready for merge**. The series fixes race conditions in Git’s HTTP transport when multiple processes concurrently fetch the same packfile URI or dumb HTTP pack, preventing corruption while preserving resumable downloads. The six-patch series is fully approved by Junio, Peff, and Taylor Blau, and will be queued to `next` imminently.

**CI image bump for static-analysis:** SZEDER Gábor’s **review of Jeff King’s patch** to bump the Ubuntu image for the static-analysis job revealed a **4.5× performance regression** in Coccinelle 1.3.1 compared to 1.1.1, directly contradicting the patch’s performance rationale. The review also identified a CI inefficiency (unnecessary build dependencies in static-analysis and documentation jobs) and suggested a follow-up cleanup.

**Gitk color-preference dialog:** Johannes Sixt’s **five-patch series overhauling gitk’s color-preference dialog** is now in a **cosmetic follow-up discussion** about phrasing refinements for UI labels. The series improves usability by making color swatches clickable, standardizing button widths, rewriting labels in natural language, and ensuring full translatability. All technical work is complete, and the discussion focuses on the phrasing of one specific label ("Diff: old lines background").

**`USE_NSEC` runtime config:** D. Ben Knoble’s **series converting `USE_NSEC` to a runtime config (`core.useNanosec`)** remains under review, with Junio identifying a **performance-critical flaw** in the core patch: the implementation calls `repo_config_get_bool()` on every invocation of `is_racy_stat()`, creating a prohibitively expensive hot-path regression. Junio proposed caching the value in the repository’s `repo_settings` struct for constant-time access.

**Use-after-free in `get_oid_with_context_1`:** René Scharfe **approved Shlok Kulshreshtha’s bugfix** for a use-after-free in `get_oid_with_context_1()` when resolving relative paths against a sparse index. The patch consolidates control flow to ensure the allocated string from `resolve_relative_path()` is freed only after its final use in `reject_tree_in_index()`. A v2 will add an `Original-patch-by:` trailer to credit Johannes Schindelin.

**`Signed-off-by:` critique:** Дилян Палаузов **opened a discussion thread** critiquing the legal and practical validity of the `Signed-off-by:` trailer, arguing that copyright law already protects published patches and that sign-off enforcement creates an uneven playing field. The thread is provocative and likely to spark debate about Git’s contribution policies and the legal assumptions behind `Signed-off-by:`.

**`git rebase -X` bug report:** zrzut01@poczta.fm **reported a usability bug** where `git rebase -X <invalid-option>` silently ignores the invalid strategy option and proceeds with the rebase, while `git merge -X <invalid-option>` correctly reports an error. The inconsistency can mislead users into believing the operation succeeded with the intended option. No patch or fix has been proposed yet.

---

## On the radar

- **`ss/repack-drop-filtered` (Siddharth Shrimali):** The series is functionally complete at v4 but awaits procedural due diligence (full test suite run and trial merges into `next`/`seen`). Junio’s proposal to replace the safety guards with a time-based heuristic remains unresolved.
- **`hn/checkout-track-fetch` (Han-Wen Nienhuys):** Marked as "Will discard" in Junio’s "What’s cooking" report, indicating it is unlikely to proceed without significant rework. The topic adds a feature to fetch the start-point branch during `git checkout --track`.
- **`tb/repack-geometric-cruft` (Taylor Blau):** Stalled in Junio’s report, with no details provided on the blocking issues. The topic combines `--geometric` and `--cruft` repack options.
- **`ps/odb-streams` (Patrick Steinhardt):** An ongoing refactoring effort to consolidate object database streaming APIs into a unified `struct odb_stream`. The topic is cooking in `seen` and is part of the broader ODB abstraction effort.