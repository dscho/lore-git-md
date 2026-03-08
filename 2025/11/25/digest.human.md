# Git Mailing List Digest - 2025/11/25

**The day in brief.** A moderately busy Tuesday with 34 emails across 13 threads, featuring significant progress on the `git replay --revert` feature, continued refinement of the `git-history` command design, and resolution of several bugfix threads. The most notable developments include Junio Hamano's detailed review of the revert functionality and Johannes Schindelin's proposed fix for the `git clone -c` whitespace regression.

## Notable threads

### `git replay` gains `--revert` capability

Siddharth Asthana introduced server-side revert functionality for `git replay`, designed primarily for GitLab's Gitaly service to avoid client-side roundtrips. The implementation cleverly reuses cherry-pick infrastructure by swapping base and pickme trees in merge operations. Junio Hamano provided extensive review feedback, suggesting code reorganization to eliminate duplication in revert message generation and requesting clearer documentation about the `--contained` incompatibility. Test failures emerged when applying the patch to current master, indicating the need for further debugging before integration. The feature represents an important expansion of `git replay`'s capabilities but will need to coordinate with Patrick Steinhardt's parallel sequencer refactoring work.

### `git-history` command design questions

Two key discussions continued around the proposed `git-history` command. Elijah Newren and Phillip Wood refined their technical alignment on branch handling strategies during rewrites, identifying an edge case where `^{OLD_COMMIT_ID}` syntax could prevent branch updates. Separately, SZEDER Gábor raised substantial concerns about the `reword` subcommand's behavior differing from interactive rebase by operating purely in-memory without checking out files. This late-stage feedback (at v6) suggests the reword functionality may need significant redesign to match user expectations around commit message editing workflows.

### `git clone -c` whitespace regression fix

Johannes Schindelin proposed a fix for the regression where `git clone -c` arguments with whitespace around keys stopped working in v2.52.0. His solution introduces a `STRING_LIST_SPLIT_TRIM_FIRST` flag to selectively trim key whitespace while leaving values untouched. Junio Hamano countered with a more targeted fix limited to `builtin/clone.c`, marking it as a temporary compatibility measure. The thread reflects Git's careful balance between maintaining backward compatibility and cleaning up historical quirks, with this particular behavior now documented for eventual deprecation.

### CI test output visibility refinements

A resolved thread about Docker CI test output saw final polish on its commit message wording. Elijah Newren improved the explanation of how test suite quality is best measured by failure diagnosis speed rather than mere pass/fail status. Johannes Schindelin confirmed the revised wording better conveys the original intent while maintaining technical accuracy. The exchange demonstrates Git's attention to both functional correctness and clear documentation, even for already-resolved issues.

### Build system artifact handling

The build system discussion concluded with Junio Hamano accepting a patch adding `scalar` to the `make strip` target while acknowledging broader questions about build system strategy remain open. Johannes Schindelin raised the possibility of abandoning CMake support entirely in favor of meson, given CMake's failure to meet Visual Studio debugging needs. This suggestion - not yet acted upon - could represent a significant shift in Git's build infrastructure direction.

## In brief

**Submodule segfault fix** -- Final polish applied to a fix for `git submodule add` crashes when processing incomplete .gitmodules entries, with Junio Hamano accepting Elijah Newren's suggested commit message improvements.

**Shallow clone edge case** -- Junio provided final review feedback on Samo Pogačnik's border commit handling fix, noting only minor terminology inconsistencies ("border" vs "boundary") remain before merging.

**Bash completion for short options** -- Wiktor Mis added tab completion for Git's short options (-C, -P, etc.), with Junio suggesting follow-up work to combine short and long option completion logic.

**Config path segfaults** -- Jeff King and Junio aligned on keeping the current `--type=path` interface while improving test coverage for `--show-scope` interactions with optional paths.

**Secret management workflows** -- The thread exploring Git filter limitations for secret redaction saw Chris Torek share his personal dotfile management approach as an alternative to smudge filters.

## On the radar

**Object database refactoring** -- Patrick Steinhardt's 14-patch series to remove `the_repository` usage saw positive review from Toon Claes, moving closer to potential merging as part of the ongoing object source management effort.

**Parallel hook processing** -- Junio's review of the hook subsystem refactoring highlighted minor documentation and code organization improvements needed for the parallel process stdin handling changes.