# Git mailing list daily digest for 2026/08/17

## The day in brief
Junio C Hamano engaged with Michael Montalbo’s data-driven ODB reorganization counterproposal, endorsing its methodology while cautioning about header-inclusion fragility. The `git history squash` series remains stalled on procedural demands, including missing sign-offs and unresolved technical feedback. A v2 patch for zsh completion now handles all global options, and the `kh/format-rev-doc-synopsis` series is ready for `next` after resolving a rendering inconsistency. The ODB alternates refactoring saw significant progress with a v3 series addressing `core.ignoreCase` timing, though a constraint with `repo_ignore_case()` remains unresolved. Junio’s "What’s cooking" report highlighted 437 commits graduated to `master` since Git 2.55, with key topics like `ps/odb-pluggable-pack-generation` awaiting rerolls.

## Notable threads

### ODB reorganization: data-driven counterproposal
[2026/08/17/20-58-27 by Junio C Hamano]
Junio C Hamano constructively engaged with Michael Montalbo’s data-driven counterproposal for reorganizing Git’s source tree. The proposal uses mechanical signals (commit history patterns and call-graph relationships) to identify cohesive subsystems, avoiding the arbitrariness of Patrick Steinhardt’s `lib/` directory approach. Junio endorsed the methodology as a "valid thought experiment" that produced intuitive results, such as the "transport" group (`connect.c`, `fetch-pack.c`, etc.), but cautioned that header-inclusion patterns may be fragile in some cases. He also proposed decoupling the `bisect` machinery from the `revision` machinery as a new reorganization target, framing it as a historical accident rather than a deliberate design choice. The discussion shifts the debate from *whether* to reorganize to *how* to do it effectively, with Junio’s endorsement suggesting openness to data-driven approaches if the methodology is refined.

### `git history squash` series: procedural stall
[2026/08/17/08-26-43 by Phillip Wood] | [2026/08/17/20-07-48 by Kristoffer Haugsbakk]
The `git history squash` series (v13) remains stalled on procedural demands from Phillip Wood, who insists Harald Nordgren include his `Signed-off-by:` trailer *above* Harald’s to reflect the chain of custody. Phillip also reiterated unresolved technical feedback, such as a ref-filter display bug in patch 3.5, which Harald has not addressed. Kristoffer Haugsbakk clarified the correct trailer ordering: Phillip’s `Signed-off-by:` should appear above his `Helped-by:`, followed by Harald’s `Signed-off-by:`. The series is technically complete and ready for integration, but Harald’s failure to engage with these procedural and technical expectations blocks progress. Junio C Hamano’s earlier "Will replace" sign-off and test suite fixup indicate intent to merge, but the missing sign-off and unresolved feedback take precedence.

### ODB alternates refactoring: v3 series and constraints
[2026/08/17/11-09-20 by Patrick Steinhardt] | [2026/08/17/17-47-16 by Jeff King]
Patrick Steinhardt posted a v3 series (5 patches) refactoring the ODB subsystem to remove lazy-loading of alternates. The series addresses the `core.ignoreCase` timing issue by reordering repository initialization (new patch 1/5) and retains case-insensitive deduplication but documents it as `NEEDSWORK`. Jeff King (Peff) acknowledged that `repo_ignore_case()` cannot be used because it relies on `repo_config_values()`, which is hardcoded to `the_repository` and would break submodule support. This constraint leaves the timing issue unresolved, though the series otherwise addresses all prior feedback. The v3 patches are queued in `seen` and represent a significant step toward simplifying the ODB codebase, but the `repo_ignore_case()` limitation may require a follow-up patch or a revised approach.

### zsh completion: v2 patch handles all global options
[2026/08/17/19-29-05 by Lutz Lengemann]
Lutz Lengemann posted a v2 patch fixing zsh completion for `git -C <path> <command>` and similar constructs. The patch now skips *all* global options (e.g., `--git-dir`, `--work-tree`, `-p`) and their arguments to set `__git_cmd_idx`, aligning with the bash implementation. Completions now work for `git -p checkout <TAB>` and `git --git-dir=<path> checkout <TAB>`, though directory completion for chained `-C` options (e.g., `git -C dir -C <tab>`) remains broken. The patch is well-motivated, addressing Junio’s architectural feedback and D. Ben Knoble’s technical concerns. The remaining gaps are documented, and the author’s responses show careful testing. This is a ready-to-merge improvement that fixes a real usability pain point.

### `kh/format-rev-doc-synopsis`: ready for `next`
[2026/08/17/18-51-47 by kristofferhaugsbakk@fastmail.com] | [2026/08/17/21-46-53 by Junio C Hamano]
Kristoffer Haugsbakk posted v4 of a two-patch documentation series resolving an AsciiDoc/AsciiDoctor rendering inconsistency. The fix replaces the code block (`----`) with an open block (`--`) for the `[synopsis]` tag, aligning with precedent (commit a34d1d53). Junio C Hamano confirmed the series is now ready for `next`, signaling the procedural blocker is resolved. The patches improve placeholder readability and consistency in `Documentation/git-format-rev.adoc` without changing behavior or on-disk formats. The series is part of the ongoing man page synopsis-style conversion effort and looks ready for integration.

### "What's cooking" report: 437 commits graduated to `master`
[2026/08/17/07-43-45 by Junio C Hamano]
Junio C Hamano’s "What’s cooking" report highlighted 437 commits graduated to `master` since Git 2.55, including:
- `jc/add-resolved`: New `git add --resolved` command to stage conflict-resolved paths while preserving unrelated local changes.
- `tn/packfile-uri-concurrency`: Fixes concurrency issues in packfile URI downloads.
- `js/sequencer-release-odb-before-commit`: Releases the ODB handle before spawning `git commit` in the sequencer, preventing file handle leaks on Windows.

Notable topics cooking in `next` or `seen`:
- `ps/odb-pluggable-pack-generation`: Makes packfile generation pluggable via ODB callbacks; awaiting reroll.
- `mm/diff-process-hunks`: Allows external processes to act as hunk providers; stalled pending further discussion.
- `hn/history-squash`: New `git history squash` subcommand; requires design clarification.

The report also noted stalled topics like `tb/repack-geometric-cruft` and discarded topics like `hn/checkout-track-fetch`.

## In brief
- **[2026/08/17/05-17-19 by Patrick Steinhardt]** Acknowledged the forward-compatibility gap in `odb_transaction_write_pack()` and agreed to use transaction flags for context-specific configuration resolution.
- **[2026/08/17/08-21-27 by Shlok Kulshreshtha]** Posted v3 of a use-after-free bugfix in `get_oid_with_context_1()` with sparse index, adopting Patrick Steinhardt’s stylistic suggestion.
- **[2026/08/17/15-43-57 by Junio C Hamano]** Queued the use-after-free bugfix for `next`, calling the solution "exactly what anybody would expect."
- **[2026/08/17/05-18-55 by Patrick Steinhardt]** Noted that `packfile_list_prepend()` suffers from the same quadratic-time inefficiency as `append()`, but Jeff King’s analysis clarified the MRU optimization mitigates its impact.
- **[2026/08/17/14-48-54 by Kristoffer Haugsbakk]** Accepted Junio’s feedback to reorder `BUG_ON_OPT_NEG(unset)` assertions in `date_cb()` and `format_nul_cb()` for defensive programming.
- **[2026/08/17/17-43-34 by Junio C Hamano]** Questioned whether the warning-and-continue behavior for valueless `submodule.active` is an improvement over the existing hard error.
- **[2026/08/17/17-21-21 by Junio C Hamano]** Challenged the design of a worktree repair patch, proposing a new function (`read_raw_gitfile()`) to return paths verbatim.
- **[2026/08/17/21-19-32 by Kenneth Lorber]** Posted an RFC patch making `git config -e` exit non-zero if the editor fails, with Junio questioning the value of changing the behavior now.
- **[2026/08/17/23-39-15 by friel@openai.com]** Added a Trace2 event to record total bytes written in each pack file, including tests for single and split packs.