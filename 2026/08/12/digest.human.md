# Git mailing list daily digest for 2026/08/12

## The day in brief

The Git project saw formal approval of a long-running feature series, a critical type-mismatch bug fix in promisor object handling, and a flurry of Windows build system refinements. Junio C Hamano and Elijah Newren advanced several topics toward integration, while contributors debated design trade-offs in transaction systems and error messaging.

## Notable threads

### Promisor object filtering series declared ready
Elijah Newren and Kristofer Karlsson formally approved the v8 series of the promisor object filtering topic, with Newren providing a `Reviewed-by` tag. Junio C Hamano asked whether the topic can be "declared victory and mark[ed] for 'next'". Kristofer Karlsson and Elijah Newren both confirmed readiness, with Karlsson stating "I would be very happy to have it advance to next" and Newren agreeing "I believe it's ready now".

### Type-mismatch bug blocks promisor object series
Junio C Hamano identified a blocking type-mismatch bug in `list_objects_filter__filter_oidset()`: the function passes the address of an `unsigned long` variable to `struct object_info.sizep`, which expects `size_t *`. Siddharth Shrimali acknowledged the bug and confirmed the fix (changing `unsigned long` to `size_t`) for v5. Christian Couder provided additional feedback on code comment formatting, punctuation, and documentation organization, suggesting squashing most of the `--drop-filtered` description into patch 5/7. Johannes Schindelin confirmed that three deferred items (remote verification, recency-based culling, drop log) are now explicitly follow-up work, not blocking requirements.

### Bash completion bugfix series advances
Junio C Hamano posted v6 of the bash completion bugfix series, now with three patches: 1/3 refactoring `_git_diff()` control flow, 2/3 implementing tracked-path-only completion for `git checkout`, and 3/3 adding a fallback to untracked paths. Elijah Newren suggested adding `--directory` to the `__git_complete_index_file "--others"` call in patch 3/3 for consistency and performance.

### Windows build system refinements posted
Johannes Schindelin posted v2 of the MinGW/Windows build adjustment series, addressing cross-build-system inconsistencies. The series removes hard-coded defaults, enables Python support, and refines compiler/linker flags. Junio C Hamano reinforced the precedent for defensive programming by pointing to existing `ifneq (,$(MSYSTEM))` guards. Johannes Schindelin dismissed the empty-`MSYSTEM` scenario as too unlikely to warrant additional build-system guards.

### Gitk color-preference dialog overhauled
Johannes Sixt posted v2 of the gitk color-preference dialog overhaul, now with six patches. The series consolidates color-setting logic, refactors repetitive Tcl/Tk code, makes color swatches clickable, updates UI labels to natural language, ensures translatability, and reorders color options. The changes aim to improve usability and maintainability of the gitk interface.

## In brief

- **ODB alternates series updated**: Patrick Steinhardt posted v2 of the ODB alternates series, addressing minor feedback and clarifying environment variable handling. Junio C Hamano reported a minor merge conflict during integration.
- **Flaky test fix advances**: Patrick Steinhardt posted v2 of the flaky test fix series, addressing minor feedback on commit message wording. The series refactors tests to use throwaway repositories and subshells.
- **`.keep` file design questioned**: Patrick Steinhardt questioned the fundamental design of ".keep" files in the new transaction system, proposing an alternative design that would avoid temporary ".keep" files.
- **Error message style debated**: Christian Couder rejected Patrick Steinhardt’s suggestion to condense an error message, arguing consistency with existing patterns. Junio C Hamano accepted the patch as-is.
- **`http.sslVerifyStatus` motivation provided**: Grayson Gordon provided concrete motivation for `http.sslVerifyStatus`, citing government customers who mandate OCSP stapling. graysongordon-gl posted v3 addressing a test numbering conflict.
- **Quadratic-time performance regression fixed**: Johannes Schindelin posted a bugfix patch addressing a quadratic-time performance regression in `packfile_store_add_pack()`. Junio C Hamano endorsed the fix as "clever and clean".
- **Valueless `promisor-remote` capability rejected**: Elijah Newren posted a bugfix patch rejecting valueless `promisor-remote` capability in Git protocol v2.
- **`git diff -l` help text revised**: Elijah Newren proposed revised wording for the `git diff -l` help text. Junio C Hamano suggested making the message more user-friendly.
- **Notes in `git log --all` discussed**: Marcel Svitalský reported unexpected pseudo-commits in a custom `git lg` alias output, later identified as Git notes. D. Ben Knoble proposed workarounds.