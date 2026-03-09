# Git Mailing List Digest - 2026/01/12

**The day in brief.** A busy Monday with 145 emails across 26 threads, dominated by two major feature series reaching completion: Adrian Ratiu's submodule gitdir path encoding (v9) and Patrick Steinhardt's git-history command (v10). Both received final approvals for merging after extensive review. Notable infrastructure work included René Scharfe's tree-parsing refactoring and Taylor Blau's MIDX corruption fix.

## Notable threads

**Submodule gitdir path encoding finalized** -- Adrian Ratiu's 11-patch series implementing runtime configuration for submodule gitdir paths via `extensions.submodulePathConfig` is approved for merging in its v9 iteration. The feature addresses filesystem conflicts through a four-stage resolution process (plain name → URL-encoded → numbered → hashed fallback) with comprehensive case-folding protection. Junio Hamano notes the series has reached "diminishing returns" for further improvements after addressing all feedback from Patrick Steinhardt and others. The implementation has been validated in production at Google, where Josh Steadmon confirms it replaces their downstream solution.

**git-history command ready for merging** -- Patrick Steinhardt's 8-patch series introducing the experimental `git history` command with initial `reword` subcommand completes its tenth iteration. The implementation builds on Git's replay infrastructure to provide fast, in-memory history editing capabilities inspired by Jujutsu. After extensive review from Elijah Newren (merge/rebase expert) and resolution of SZEDER Gábor's forward-compatibility concerns, the series is now merge-ready. The command works with dirty working trees and updates all local branches containing rewritten commits by default, though currently skips hook execution.

**Ref backend validation unified** -- Patrick Steinhardt's 17-patch series to centralize ref consistency checks across files and reftable backends concludes with all review feedback addressed. The work moves fsck checks from `builtin/fsck.c` into the refs subsystem, introducing shared validation functions (`refs_fsck_ref()`, `refs_fsck_symref()`) that work identically for both storage formats. Karthik Nayak (reftable co-author) confirms the implementation properly handles worktree edge cases that his original code missed. The series establishes critical infrastructure for future ref backend work while maintaining backward compatibility.

**Tree parsing freed from the_repository** -- René Scharfe's 10-patch series completes the tree-parsing portion of the multi-year `the_repository` removal effort. The work introduces repository-aware variants of tree parsing functions (`repo_parse_tree_gently()` etc.) and systematically converts callers across subsystems. A semantic patch automates conversion of remaining calls in 27 files. The only outstanding discussion concerns deprecation timeline documentation, where Junio Hamano reaffirms the project's policy against version-specific removal markers in favor of technical readiness.

**git status shows push tracking** -- Harald Nordgren's long-running series (23 iterations) to enhance `git status` with push tracking branch divergence is approved for `next`. The implementation cleanly extends the status output to show both upstream and push tracking when they differ, maintaining all existing behavior including quick mode (`--no-ahead-behind`). Phillip Wood's review refined the advice display logic to be consistent across tracking modes. The changes affect core status formatting but come with 262 lines of new test coverage in t6040.

## In brief

**HTTP authentication fix for probe_rpc** -- Patrick Steinhardt confirms Aaron Plattner's test approach for fixing Bearer token authentication in remote-curl during filtered clones. The 2-line fix ensures credentials are included in probe_rpc() calls.

**MacOS iconv workaround** -- Two-part series adds ICONV_RESTART_RESET workaround for macOS 14/15's ISO-2022-JP conversion bug, with build system support to auto-enable when needed.

**Test modernization for post-checkout hook** -- Deveshi Dwivedi's merged series improves t5403 with a `check_post_checkout` helper and `test_cmp` validation, following patterns from t5510/t5520.

**MIDX corruption handling fix** -- Taylor Blau addresses an edge case where MIDX writes could skip updates for corrupt files, splitting the fix from his larger compaction work for v2.53.0 inclusion.

**Documentation conversion** -- Michael Lyons updates git-bisect.adoc to synopsis style, following Jean-Noël Avila's standardization effort with glossary term conversions.

**MyFirstContribution guide improvements** -- Shreyansh Paliwal's documentation patch adds missing header includes and clarifies build prerequisites after incorporating Junio's feedback about dependency specificity.

## On the radar

**Pluggable ODB backends** -- Patrick Steinhardt's object info handling refactoring (v5) is queued with 9% disk size lookup speedup and all review feedback addressed.

**Windows symlink support prep** -- Johannes Schindelin and Karsten Blees' series addressing mingw compatibility and path handling is approved for `next`.

**repo-info keys feature** -- Lucas Seiki Oshiro's `--keys` flag for programmatic repository metadata access awaits final wording tweak about output format naming.

**Template file tracking proposal** -- Krzysztof Zabłocki's `.gittemplates` idea sparks discussion about working tree synchronization challenges, with brian m. carlson noting design constraints.