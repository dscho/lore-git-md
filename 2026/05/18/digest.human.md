# Here's the Git mailing list daily digest for May 18, 2026:

## The day in brief

A moderately busy Monday with 73 emails across 17 threads, featuring several patch series reaching completion and important technical discussions. Key highlights include Derrick Stolee's negotiation control series getting final approval, Patrick Steinhardt's `the_repository` removal work progressing, and substantive reviews of Harald Nordgren's branch pruning and checkout tracking features.

## Notable threads

### Remote group push support finalized

Usman Akinyemi's series adding remote group support to `git push` (mirroring existing fetch functionality) is now complete after six iterations. The v6 patches include only minor documentation formatting fixes from Kristoffer Haugsbakk's review. Key aspects:
- Each remote is pushed independently via child processes for failure isolation
- Refreshes per-remote refspec lists to handle `remote.<name>.push` configs
- Explicitly rejects `--atomic` for group pushes
- Comprehensive test coverage in t5566-push-group.sh

The series appears ready for merging with all technical feedback addressed.

### Negotiation controls for fetch/push

Derrick Stolee's 8-patch series introducing `--negotiation-include` and `--negotiation-restrict` options has received final approval from reviewer Matthew Cheetham. The feature addresses inefficient object transfer in monorepos by providing precise control over which refs participate in negotiation. The v5 version:
- Adds config support via `remote.<name>.negotiationInclude`
- Completes push integration when `push.negotiate` is enabled
- Uses new `have_sent()` negotiator interface for cleaner implementation
- Maintains comprehensive test coverage

All patches now carry Reviewed-by tags and the series is ready for maintainer consideration.

### `the_repository` removal in setup subsystem

Patrick Steinhardt's 18-patch series converting setup.c functions to use explicit repository parameters has progressed to v2, addressing thread-safety concerns from Junio Hamano and Tian Yuchen. The changes:
- Remove static buffers in `is_inside_worktree()` and `is_inside_git_dir()`
- Eliminate `initialized` flag from `setup_work_tree()`
- Systematically convert all setup-related functions
- Maintain identical behavior while removing global state

The series represents a major chunk of the multi-year effort to eliminate `the_repository` global variable.

### Checkout tracking with automatic fetch

Harald Nordgren's `--track=fetch` feature for `git checkout`/`git switch` has reached v10 with all technical feedback addressed. The patch:
- Automatic fetching when creating tracking branches
- Correct handling of hierarchical remote names
- Comprehensive test coverage (16 test cases)
- Maintains backward compatibility

While Junio Hamano's philosophical objection to the workflow remains, his silence on the technical refinements suggests potential acceptance.

## In brief

**Date approximation fixes** -- Tuomas Ahola follows up his approxidate series with a test reorganization patch, grouping related "noon yesterday" test cases together after the main series was accepted.

**Graph log root commit visualization** -- Pablo Sabater presents final design comparisons for displaying root commits in `git log --graph`, weighing indentation versus blank-line approaches before maintainer decision.

**Config file locking timeout** -- Junio Hamano and Patrick Steinhardt discuss implementation details for Jörg Thalheim's configurable timeout, debating process-wide versus per-repository behavior.

**Partial clone prefetch optimization** -- Elijah Newren's series for `git cherry` and `git grep` is merged to `next` after Derrick Stolee confirms the command-specific prefetch architecture is appropriate.

**Octopus merge optimization** -- Kristofer Karlsson clarifies his patch simplifying the up-to-date check was primarily for code clarity rather than measurable performance gains.

**Word-diff whitespace documentation** -- Michael Montalbo revises his documentation patch based on Junio Hamano's feedback, focusing on user expectations rather than internal mechanics.

**strbuf overflow optimizations** -- René Scharfe's v2 series improves overflow checking patterns, adding Clang-specific optimizations that show 4% speedup on ARM64.

**Commit fixup message options** -- Erik Cervin-Edin proposes allowing `-m` and `-F` with `--fixup=amend/reword`, receiving detailed reviews from Junio Hamano and Phillip Wood about implementation and test style.

## On the radar

**Branch pruning interface** -- Phillip Wood raises design questions about Harald Nordgren's `--prune-merged` regarding argument handling and remote specification, though the core functionality is approved.

**Documentation conversion** -- Jean-Noël Avila's man page standardization work hits a snag as Junio Hamano questions overuse of [synopsis] formatting in examples, particularly with shell prompts.

**Rebase segfault diagnosis** -- Phillip Wood suggests a reported rebase crash during signing may relate to known pack corruption from parallel maintenance operations rather than the apparent NULL-dereference.