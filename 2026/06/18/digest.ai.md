# The Git Mailing List Daily Digest for 2026/06/18

**The day in brief.** Thursday was a busy day on the Git mailing list, with 98 emails across 28 threads. The standout developments were Harald Nordgren's `git branch --delete-merged` series reaching completion (v16), Junio C Hamano merging Patrick Steinhardt's Windows large-object handling series, and Karthik Nayak's line-log enhancements receiving maintainer feedback. Several long-running threads—including Pablo Sabater's `git log --graph` visualization series and Harald's `--track=fetch` feature—saw incremental progress, while a new RFC on "pinned references" sparked early design discussion.

---

## Notable threads

### `git branch --delete-merged` reaches completion
**Harald Nordgren's 16-iteration series adding safe automated branch cleanup landed in v16**, addressing all prior feedback. The command deletes local branches whose work has been merged into their upstream, with multiple safety checks: it never deletes checked-out branches, skips branches whose upstream no longer exists, preserves branches that push back to their upstream (like `main` tracking `origin/main`), and supports per-branch opt-out via `branch.<name>.deleteMerged` config. The `--dry-run` option provides a preview of deletions.

Junio C Hamano's earlier skepticism about protecting branches that serve as upstreams for other branches was resolved in favor of simplicity, with the maintainer concluding the added complexity wasn't justified for typical rebase workflows. Phillip Wood's high-weight feedback on API consistency and test structure was fully addressed, and the series is now ready for integration. The implementation touches `builtin/branch.c`, adds comprehensive test coverage, and establishes a foundation for future branch management improvements.

---

### Windows large-object handling series merged
**Junio C Hamano merged Johannes Schindelin's 8-patch series converting `unsigned long` to `size_t` for object sizes** across delta handling, pack-objects, packfiles, and ODB subsystems. The series resolves 4GB+ object support on Windows, where `size_t` is 64-bit but `unsigned long` is 32-bit. Key changes include updating the MSVC `ftruncate()` compatibility layer to use `_chsize_s` for 64-bit sizes and introducing a new `get_delta_hdr_size_sz()` helper.

Patrick Steinhardt confirmed all his v1 feedback was addressed in v2, and Junio's final sign-off ("nothing iffy") signals the series is ready for `next`. The changes are mechanical but wide-ranging, touching 72 files, and represent a critical step toward full large-object support on all platforms. The series also fixes test failures in `t7508` and `t5608` with 4GB+ objects.

---

### Line-log enhancements receive maintainer feedback
**Karthik Nayak's 7-patch series extending `git log -L` to support range-scoped diff stat, whitespace checking, and `-G` pickaxe** received a surface-level review from Junio C Hamano. The series builds on recent line-log cleanup work to make `--stat`, `--numstat`, `--shortstat`, `--check`, and `-G` honor tracked line ranges, so only changes or errors within those ranges are reported.

Junio's feedback focused on documentation wording, suggesting "count only lines within the tracked range" instead of the jargon "range-scoped counts." The implementation itself was praised for its conciseness, with the maintainer noting the integration of the line-range filter with `diffstat_consume()` required only a small conditional block. The series touches `diff.c`, `diffcore-pickaxe.c`, `revision.c`, and `xdiff-interface.c`, and includes extensive test coverage in `t/t4211-line-log.sh`. No technical objections were raised, and the series appears on track for merging pending minor documentation tweaks.

---

### `git log --graph` visualization series advances
**Pablo Sabater's v5 patch series improving `git log --graph` rendering of commits with excluded parents** saw incremental progress, with Jeff King (Peff) engaging on the critical prio-queue interaction blocker. The series implements cascading indentation for visual roots (except the first) while preserving parent-child relationships, but the lookahead logic in `graph_peek_next_visible()` conflicts with the dual-queue traversal model in `revision.c`.

Peff clarified that the prio-queue refactoring (commit `dd4bc01c0a`, now in `master`) limits lookahead to the first entry, making the current implementation incompatible. He asked whether the three key flags (`is_next_visible`, `next_has_column`, `is_next_visual_root`) can be set correctly without full in-order traversal, and suggested refactoring the loop body into a helper function to avoid code duplication. The discussion remains focused on whether the limited lookahead will introduce redundant indentations or other artifacts. The series is not yet ready for merging, but the technical path forward is clearer.

---

### `--track=fetch` feature stalls on workflow merit
**Harald Nordgren's v14 series extending `git checkout` and `git switch` with `--track=fetch`** remains in limbo, with the maintainer's final decision on the feature's workflow merit still outstanding. The series adds a `fetch` mode to the `--track` option that automatically fetches the remote branch before checking it out, addressing the pain point where users must manually fetch before creating a tracking branch.

Junio C Hamano has expressed sustained reservations about the feature's value, specifically whether the convenience of automatic fetching during checkout justifies the potential risks (e.g., network operations without user preview, encouraging risky workflows). Harald's outreach to Phillip Wood and D. Ben Knoble yielded no new advocacy, with Knoble suggesting that keeping fetch and checkout as separate actions might be preferable. The series is technically complete and approved, but its inclusion hinges on Junio's assessment of whether the benefits outweigh his concerns.

---

### Ref backend refactoring series sees design debate
**Patrick Steinhardt's 8-patch refactoring series modernizing Git's reference backend infrastructure** saw renewed discussion on patch 7/8, which resolves recursive initialization during "onbranch" config evaluation. Jeff King (Peff) challenged the fundamental design, arguing that `ref_store_init()` should not rely on the general config subsystem for backend-critical settings like `logAllRefUpdates`. Instead, he proposed either deferring these settings until after ref store initialization or reading them alongside repository-format settings via `read_repository_format()`, which intentionally ignores includes.

The critique suggests the current workaround—temporarily setting `ref_storage_format` to `REF_STORAGE_FORMAT_UNKNOWN`—treats a symptom rather than the root cause. Justin Tobler's earlier proposal to add an `initialized` boolean to `struct ref_store` was dismissed as insufficient, as the recursion occurs *during* `ref_store_init()` before the ref store is fully constructed. The discussion remains open, with the series otherwise uncontroversial and ready for integration pending resolution of this design question.

---

### RFC: Pinned references
**Erik Östlund proposed a new Git concept called "pinned references"**—a way to specify a reference (e.g., a tag) together with an expected object ID, ensuring the ref exists, resolves, and points to the exact OID given. The strawman syntax is `refs/tags/v1.2.3?oid=a1b2c3d4`, though alternatives are welcome.

The motivation is to combine the human readability of refs with the immutability of object IDs, creating a stable reference that external tooling doesn't have to enforce. Use cases include deployment systems and CI/CD pipelines where a tag's immutability is critical but its human-readable name is also needed. No prior art exists in Git for this "ref plus expected OID" invariant, and the proposal is purely conceptual at this stage. Likely points of contention include syntax, use-case validation, and whether this could encourage users to rely on OIDs in ways that break when objects are garbage-collected.

---

## In brief

> **`git history squash` queued in `next`** -- Harald Nordgren's series folding commit ranges into a single commit was merged after Junio C Hamano made trivial adjustments (renumbering the test script and replacing `grep` with `test_grep`). The implementation performs a single three-way merge of the range against the oldest commit's parent and replays descendants on top, avoiding the repeated conflict stops of a rebase-based approach.

> **Windows CI reliability improvements** -- Patrick Steinhardt's PowerShell script for direct dependency management in GitLab CI was confirmed ready to merge. The script reduces setup time from ~3 to ~1 minute and improves reliability by eliminating Chocolatey overhead. Justin Tobler's review confirmed all prior feedback was addressed.

> **Zsh completion bugfix** -- Lutz Lengemann's patch fixing `git -C <path> <command>` completion in zsh received substantive review from D. Ben Knoble, who identified limitations in the variable propagation and directory completion logic. The patch addresses the immediate issue but may require further iteration.

> **`ignore_case` libification** -- Justin Tobler and Tian Yuchen's series moving the global `ignore_case` variable into `struct repo_config_values` was approved by Junio C Hamano after aligning the getter's logic with the existing pattern for other repository-specific configuration getters. The series touches 14 files and continues the `the_repository` removal effort.

> **Interactive typo correction** -- A patch enhancing Git's command typo handling to prompt interactively with "Did you mean 'git checkout neo'? [y/N]" was questioned by Justin Tobler, who noted the behavior already exists via `help.autoCorrect=prompt`. The patch's necessity is now in doubt.

> **Documentation updates** -- Several threads saw progress on contributor documentation:
> - Kristoffer Haugsbakk's series updating `SubmittingPatches` for trailer conventions was merged, with a post-merge follow-up addressing AsciiDoc anchor placement.
> - Michael Montalbo's patches clarifying design critique expectations in `SubmittingPatches` received maintainer approval.
> - Junio C Hamano's patch requiring design justification in resubmissions sparked discussion about the project's "show me the code" norm and contributor accessibility.

> **Test suite fixes** -- Todd Zullinger's report of TAP parse errors in `t4216-log-bloom.sh` on non-x86 architectures was resolved by removing the `SIGNED_CHAR_BY_DEFAULT` prerequisite, which was unused and redundant. The fix addresses a long-standing issue exposed by a recent TAP harness strictness change.

---

## On the radar

- **Pablo Sabater's `git log --graph` visualization series** remains blocked on the prio-queue interaction issue. The next step is for Pablo to demonstrate whether the three key flags can be set correctly with the limited lookahead.
- **Harald Nordgren's `--track=fetch` feature** awaits Junio C Hamano's final assessment of whether the workflow benefits justify its inclusion. The series is technically complete but lacks broad support for its premise.
- **Patrick Steinhardt's ref backend refactoring series** is stalled on the design question for patch 7/8. Jeff King's critique may require a more invasive refactoring than the current incremental approach.
- **Erik Östlund's pinned references RFC** is in the early design phase. Follow-up discussion may focus on syntax, use-case validation, and implementation feasibility.