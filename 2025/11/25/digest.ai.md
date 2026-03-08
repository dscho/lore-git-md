# Git Mailing List Digest - 2025/11/25

**The day in brief.** A moderately busy Tuesday with 34 emails across 13 threads, featuring significant progress on the `git replay --revert` feature, continued discussion of the `git-history` command design, and resolution of several bugfix threads. The most notable developments include Junio Hamano's detailed review of the server-side revert functionality and Johannes Schindelin's proposed fix for the `git clone -c` whitespace regression.

## Notable threads

### `git replay` gains `--revert` capability

Siddharth Asthana introduced server-side revert functionality to `git replay`, adding a `--revert` flag that mirrors `git revert` but operates directly on bare repositories. The implementation cleverly reuses cherry-pick infrastructure by swapping base and pickme trees in merge operations. Johannes Schindelin suggested considering `git merge-tree` for bulk reverts of entire Merge Requests, while Junio Hamano provided extensive review feedback focusing on code organization and documentation clarity. Test failures emerged when applying the patch to current master, indicating the need for further refinement before merging.

### `git-history` command design refinements

The proposed `git history` command saw two key discussions today. Elijah Newren and Phillip Wood continued their technical exchange about branch handling strategies during history rewriting, identifying an edge case where `--ancestry-path` combined with `--branches` would fail to update branch tips. Separately, SZEDER Gábor raised important concerns about the `reword` subcommand's behavior differing from interactive rebase by operating purely in-memory without checking out files - a substantial usability gap for non-trivial commit message edits. Both discussions highlight tensions between the command's goal of lightweight history editing and maintaining expected workflow capabilities.

### `git clone -c` whitespace regression fix

Johannes Schindelin proposed a fix for the regression where `git clone -c` arguments with whitespace around keys stopped working in v2.52.0. His solution introduces a `STRING_LIST_SPLIT_TRIM_FIRST` flag to selectively trim whitespace from keys while leaving values untouched. Junio Hamano later suggested a more targeted fix specific to `builtin/clone.c`, marking it as a temporary compatibility measure with a NEEDSWORK comment. The thread reflects Git's pragmatic approach to backward compatibility with real-world usage while maintaining clean design principles.

### CI test output visibility refinements

The thread about CI test output visibility in Dockerized jobs reached its conclusion today with final polish on the commit message wording. Elijah Newren's suggested clarification - that test suite quality is best measured by failure diagnosis speed rather than just pass/fail status - was accepted by both Johannes Schindelin and Junio Hamano. This resolved thread demonstrates the project's attention to both technical correctness and clear documentation of the underlying philosophy.

### Build system artifact handling

Johannes Schindelin and Junio Hamano discussed build system consistency in artifact handling across Makefile, CMake and meson. Schindelin suggested potentially abandoning CMake support entirely given meson's adoption as the modern build system, marking a notable shift in build strategy. The immediate Makefile restructuring around `OTHER_PROGRAMS` vs `OTHER_ARTIFACTS` appears ready for merging, while the broader build system questions remain open for future discussion.

## In brief

**Submodule segfault fix** -- Junio Hamano finalized a bugfix for `git submodule add` crashes when processing incomplete .gitmodules entries, accepting Elijah Newren's suggested commit message improvements.

**Shallow clone edge case** -- Junio reviewed Samo Pogačnik's v2 patch fixing border commit handling in shallow clones, noting only minor terminology cleanup ("border" vs "boundary") remains before merging.

**Config path segfaults** -- Jeff King and Junio discussed interface design for `git config --type=path` with `:(optional)` prefixes, agreeing to keep the current NULL-based approach while adding test coverage for `--show-scope` interactions.

**Bash completion for short options** -- Wiktor Mis added tab completion for Git's short options (-C, -P, etc.), with Junio suggesting follow-up work to combine short and long option completion.

**Secret management workflows** -- The thread about Git filter limitations for secret management explored alternative approaches, with Chris Torek sharing his dotfile template workflow that avoids smudge filter constraints.

## On the radar

**Object database refactoring** -- Patrick Steinhardt's 14-patch series refactoring object source management received positive review from Toon Claes, moving closer to potential merging as part of the ongoing effort to remove `the_repository` usage.

**Hook subsystem changes** -- Junio provided technical feedback on the parallel process stdin handling in the hook refactoring series, suggesting improved code organization but generally approving the approach.

**Xdiff cleanup** -- Elijah Newren confirmed the extensive `en/xdiff-cleanup-2` refactoring is ready for merging despite being part of a larger ongoing effort, with more work needed in future series.