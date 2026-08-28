# Git mailing list daily digest for 2026/08/27

## The day in brief
The `--autosquash` fix for empty commits is ready for final review, while the trace2 hardening series faces deeper architectural questions. A `git checkout -m` autostash series is contested on performance grounds, and the ODB fsck pluggability series receives light review. Junio proposed a major architectural shift for the `the_repository` removal effort, replacing `repo` parameters with a simple `bool` flag in built-in commands.

## Notable threads

### `--autosquash` fix for empty commits (2026/07/10/04-13-11)
Farid Zakaria’s bugfix for `--autosquash` behavior when `fixup!`/`squash!` commits cancel out their target is now in its fourth iteration (`v4`) and ready for final review. The patch introduces `is_amended_head_empty()` to detect when applying a `fixup!` or `squash!` empties the target commit, ensuring `--empty` is honored for such commits while preserving existing behavior for workflows that rely on empty commits as placeholders. The implementation handles edge cases like mid-chain fixups and conflict resolution, and includes comprehensive test coverage.

Today’s update confirms the v4 link on lore.kernel.org, resolving the last blocker. The patch touches `sequencer.c`, `Documentation/git-rebase.adoc`, and adds tests in `t/t3415-rebase-autosquash.sh` and `t/t5407-post-rewrite-hook.sh`. The follow-up discussion about a potential `revert!` directive remains unresolved but is orthogonal to this patch.

### Trace2 hardening against `die()` (2026/07/15/16-12-11)
Jeff King (Peff) expanded his critique of Derrick Stolee’s trace2 hardening series, arguing that the current approach is a “tip of the iceberg” that may require a ground-up rewrite of trace2. The series replaces `die()`-triggering helpers with defensive fallbacks, but Peff’s review highlights that indirect calls via helpers like `strbuf` or `json-writer.c` could still crash Git. Elijah Newren agreed to drop patch 3/4 entirely, replacing it with a targeted fix for `git mktree --batch` that removes the `OBJECT_INFO_QUICK` flag while retaining `SKIP_FETCH_OBJECT`.

The core fix (patch 4/4) remains ready for merging, having adopted Peff’s tri-state design for `fill_midx_entry()`. The series touches 17 files within the trace2 subsystem and introduces `banned-die.h` as a new architectural enforcement mechanism. The discussion now centers on whether the hardening effort should expand to address indirect dependencies or accept the current approach as a pragmatic first step.

### `git checkout -m` autostash conflict handling (2026/07/25/15-34-27)
Phillip Wood contested the performance trade-off of the fast-path optimization in Harald Nordgren’s two-patch series refining `git checkout -m` autostash behavior. The optimization skips the autostash retry loop when no tracked changes exist, but Phillip argues the pre-check overhead outweighs the benefit of avoiding occasional redundant stash cycles. He suggests a deeper refactoring of `unpack_trees()` would be needed to make the optimization truly effective.

The series also makes the sequencer’s autostash apply logic explicitly report conflicts via a `conflicted` output parameter, but Phillip objects to this API design, preferring to repurpose the return value instead. Junio had previously accepted patch 2/2 conditionally, but Phillip’s feedback signals the series is unlikely to proceed without revision. The improved advice formatting in patch 2/2 remains uncontroversial.

### ODB fsck pluggability (2026/08/25/14-30-02)
Karthik Nayak provided light review of Patrick Steinhardt’s 10-patch series making ODB fsck checks pluggable. Feedback was limited to minor wording and spelling nits, with one substantive question about the use of `OPT_BIT` versus `OPT_BOOL` for the `--full` flag. The series is functionally complete, with no remaining technical objections, and prepares the codebase for future pluggable ODB backends.

The patch moves fsck logic from `builtin/fsck.c` into backend-specific implementations (e.g., `odb/source-packed.c`, `odb/source-loose.c`), replacing dedicated error bits (`ERROR_PACK`, `ERROR_BITMAP`) with the generic `ERROR_OBJECT`. The series touches 10 files and includes test coverage for the new backend-specific logic.

### `the_repository` removal effort (2026/08/27/18-29-00)
Junio C Hamano proposed a major architectural shift for the `the_repository` removal effort, submitting a patch to replace the `struct repository *repo` parameter in built-in commands with a simple `bool has_repo` flag. The patch touches 135 files and converts every built-in command’s signature, arguing that the `repo` parameter is misleading and encourages wasted effort converting code that will never be libified.

The proposal addresses segfault risks in edge cases like `cd / && git foo -h` and reaffirms the distinction between built-in commands (repository-specific) and general utility code (repository-agnostic). Junio’s patch is tested and passes the test suite, but the community has not yet weighed in on the architectural shift. The thread also clarified that built-in commands are not intended for direct reuse via libgit, redirecting the `the_repository` removal effort toward utility code outside `builtin/`.

## In brief
- **`git worktree add` ambiguous branch error messages** (2026/08/08/08-21-41): Junio marked the v13 iteration as ready for `next`, with no further changes expected.
- **Worktree repair path resolution** (2026/08/15/13-11-19): Junio accepted the functional split between `read_gitfile_raw()` and `read_gitfile_gently()`, leaving only commit-message clarification as the remaining action.
- **Geometric repacking race condition** (2026/08/18/22-34-04): No new developments; the series remains under active debate.
- **Branch protection from deletion** (2026/08/25/21-25-15): Elijah Newren identified a logical gap: the protection is not transitive, despite documentation claiming it applies to “directly or indirectly” upstream branches.
- **Unsafe commit operations during conflict resolution** (2026/08/26/05-21-21): Junio questioned the necessity of blocking `git commit <paths>` during conflict resolution for empty commits, introducing uncertainty about the series’ direction.
- **Deprecation warning rewording** (2026/08/26/14-33-31): Junio resolved Elijah Newren’s grammatical nit in the v3 patch, which is now cooking in `next`.
- **`die_for_incompatible_opts()` helper** (2026/08/26/23-31-50): Junio posted a v2 patch implementing the `EOF` sentinel design and fixing the `va_end()` omission.
- **French `git-checkout` man page** (2026/08/27/12-53-02): Eric Gautier reported a grammatical error in the French translation, which Ben Knoble escalated to the French translation maintainers.
- **`git-svn` migration noise and build option** (2026/08/27/16-51-32): Wesley Schwengle posted a v2 patch fixing a typo in the error message.