# Git mailing list daily digest for 2026/08/12

## The day in brief

Elijah Newren and Kristofer Karlsson confirmed readiness for a long-running series to advance to `next`, while Junio C Hamano identified a blocking type-mismatch bug in a promisor remote series. Johannes Schindelin posted a comprehensive MinGW/Windows build adjustment series, and multiple contributors engaged in design discussions about transaction systems, ODB alternates, and HTTP security options.

## Notable threads

### Promisor remote series declared ready for `next`
[2026/06/20/10-36-53]

Elijah Newren formally approved the v8 series with `Reviewed-by: Elijah Newren <newren@gmail.com>`, confirming all prior feedback was addressed. Kristofer Karlsson acknowledged the series’ readiness and endorsed the final state as consistent and well-documented. Junio C Hamano explicitly asked whether the topic can be "declared victory and mark[ed] for 'next'". Kristofer Karlsson and Elijah Newren both confirmed readiness for advancement.

### Promisor remote partial clone series blocked by type-mismatch bug
[2026/07/14/22-48-33]

Johannes Schindelin confirmed the three deferred items (remote verification, recency-based culling, drop log) are now explicitly follow-up work, not blocking requirements for this series. Christian Couder provided surface-level feedback on code comment formatting, punctuation, and grammar in `enumerate_promisor_blobs()`, `builtin/repack.c`, and the test script. Christian Couder also suggested making the `opts` parameter in `list_objects_filter__filter_oidset()` `const` and proposed reorganizing the documentation by squashing most of the `--drop-filtered` description into patch 5/7.

Junio C Hamano identified a blocking type-mismatch bug in `list_objects_filter__filter_oidset()`: the function passes the address of an `unsigned long` variable to `struct object_info.sizep`, which expects `size_t *`. Siddharth Shrimali acknowledged the bug and confirmed the fix (changing `unsigned long` to `size_t`) for v5. Siddharth Shrimali declined to reuse `die_if_some_operation_in_progress()` due to refactoring effort, proposing to improve the error message instead.

### Bash completion bugfix series advances to v6
[2026/08/03/00-58-01]

Junio C Hamano posted v6 of the bash completion bugfix series, now with three patches: 1/3 refactored `_git_diff()` control flow, 2/3 implemented tracked-path-only completion for `git checkout` using `__git_complete_index_file ""`, and 3/3 added a fallback to untracked paths when no tracked paths match, using `__git_complete_index_file "--others --directory"`. Elijah Newren suggested adding `--directory` to the `__git_complete_index_file "--others"` call in patch 3/3 for consistency and performance.

### Transaction system design questioned
[2026/07/16/13-28-41]

Patrick Steinhardt questioned the fundamental design of ".keep" files in the new transaction system, proposing an alternative design that would write the packfile to the temporary directory without a ".keep" file, then write a ".keep" file to the main object directory at commit time before migrating the packfile, and remove the ".keep" file during finalization.

### MinGW/Windows build adjustments posted as v2
[2026/08/05/14-26-26]

Johannes Schindelin dismissed the empty-`MSYSTEM` scenario as too unlikely to warrant additional build-system guards. Johannes Schindelin posted v2 of the MinGW/Windows build adjustment series, addressing the cross-build-system inconsistency in Patch 8 by aligning Meson’s behavior with the Makefile’s. The series includes 12 patches: enabling Python support, removing hard-coded `CC = gcc`, removing `-D_USE_32BIT_TIME_T`, refining linker flags, removing redundant entry-point specifications, replacing hard-coded paths with dynamic settings, making MSYS2-specific code conditional, replacing hard-coded defaults, disabling hard-linked executables, simplifying path logic, fixing a locale-handling regression, and allowing direct `git.exe` invocation. Junio C Hamano reinforced the precedent for defensive programming by pointing to existing `ifneq (,$(MSYSTEM))` guards in `config.mak.uname`.

### gitk color-preference dialog overhaul posted as v2
[2026/08/06/06-40-13]

Johannes Sixt posted v2 of the gitk color-preference dialog overhaul, now with six patches: 1/6 consolidated the `prefspage_set_colorswatches` helper procedure, 2/6 refactored repetitive Tcl/Tk code into a single `foreach` loop, 3/6 made color swatches clickable by replacing `ttk::button` widgets, 4/6 updated seven UI labels to use more natural language, 5/6 ensured full translatability by replacing dynamically constructed dialog titles, and 6/6 reordered the color options list.

### Flaky test fix series advances to v2
[2026/08/07/04-02-43]

Karthik Nayak suggested documenting the switch from `test_config` to `git config set` in the commit message and confirmed the refactoring approach as correct. Patrick Steinhardt explained that the switch was forced by `test_config`’s reliance on `test_when_finished`, which cannot run in subshells. Patrick Steinhardt posted v2 of the flaky test fix series, addressing minor feedback on commit message wording. Patch 1/2 refactored three tests to use throwaway repositories and subshells, and Patch 2/2 fixed the flake by restoring default auto-detach behavior.

### ODB alternates series advances to v2
[2026/08/10/13-33-27]

Patrick Steinhardt defended retaining the hashmap-based deduplication as a "safe side" measure and confirmed that alternates are fully initialized during `odb_new()`. Patrick Steinhardt posted v2 of the ODB alternates series, addressing minor feedback and adding a comment to clarify environment variable handling. The series includes four patches: decoupling source path comparisons from `the_repository`, eagerly initializing alternates during ODB setup, and removing now-unused fields. Junio C Hamano reported a minor merge conflict during integration and asked Patrick Steinhardt to verify the synthesized merge result.

## In brief

- **[2026/08/07/10-59-00]** Yoichi Nakayama proposed omitting warnings/advice in the `--guess-remote` case, as the command is designed to succeed even if the guess fails. Junio C Hamano proposed an alternative implementation extending `unique_tracking_name()` to optionally return conflicting remotes.
- **[2026/08/10/19-04-48]** Johannes Schindelin posted v2 of the sequencer ODB release fix, addressing Junio C Hamano’s request for clearer commit-message rationale.
- **[2026/08/11/03-19-32]** Patrick Steinhardt suggested condensing an error message, but Christian Couder rejected the suggestion for consistency. Junio C Hamano accepted the patch as-is.
- **[2026/08/11/12-14-46]** Patrick Steinhardt clarified that `http.sslVerifyHost` can be used securely with certificate pinning.
- **[2026/08/11/17-02-00]** Patrick Steinhardt questioned the practical value of `http.sslVerifyStatus`. Grayson Gordon provided concrete motivation citing government customers. graysongordon-gl posted v3 addressing a test numbering conflict.
- **[2026/08/12/06-39-43]** Elijah Newren posted a bugfix patch rejecting valueless `promisor-remote` capability in Git protocol v2.
- **[2026/08/12/06-42-37]** Elijah Newren posted a documentation patch correcting the short help message for `git diff -l`, with Junio C Hamano suggesting less precise wording.
- **[2026/08/12/06-42-38]** Elijah Newren posted a cleanup patch removing an unused variable assignment in `sequencer.c`.
- **[2026/08/12/12-31-45]** Marcel Svitalský reported unexpected pseudo-commits in a custom `git lg` alias output, identified as Git notes by Kristoffer Haugsbakk. D. Ben Knoble proposed workarounds.
- **[2026/08/12/15-55-56]** Johannes Schindelin posted a security bugfix patch restricting advertised bundle URIs by protocol allow-list.
- **[2026/08/12/17-59-59]** Elijah Newren posted a `.mailmap` update mapping his current and previous work addresses.
- **[2026/08/12/19-11-09]** Johannes Schindelin posted a bugfix patch addressing a quadratic-time performance regression in `packfile_store_add_pack()`. Junio C Hamano endorsed the fix as "clever and clean", while Jeff King questioned the value of the new perf test.