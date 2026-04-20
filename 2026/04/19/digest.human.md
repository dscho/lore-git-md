# Git Mailing List Digest - 2026/04/19 (Sunday)

**The day in brief.** A moderately active Sunday with 21 emails across 13 threads, featuring substantive technical discussions on several fronts. The most notable developments include Ian Jackson's Rust/libgit2 prototype for `git subtree split` as an alternative to shell script fixes, finalization of Deveshi Dwivedi's `git stash` argument parsing improvements, and continued refinement of Derrick Stolee's fetch/push controls. Localization updates for Git 2.54.0 also progressed.

## Notable threads

### Rust prototype for `git subtree split` emerges

Ian Jackson escalated the long-running `contrib/subtree` discussion by presenting a working Rust/libgit2 prototype as an alternative to Colin Stagner's shell script fixes. The prototype demonstrates performance improvements but proposes changing committer information handling - using dummy committers with original attribution in messages. This shifts the architectural debate from theoretical to practical, raising questions about whether the solution should live in git.git (with its limited Rust infrastructure) or as a standalone tool. The thread now explicitly weighs three options: Colin's shell fix, Ian's Rust rewrite as a standalone tool, or integration into git.git.

### `git stash` argument parsing finalized

Deveshi Dwivedi's patch to improve `git stash` argument parsing reached its final form, simplifying the logic to unconditionally assume the "push" subcommand when the command line starts with an option flag. The implementation now checks for any leading dash (`-`) to trigger the push assumption, making it more maintainable as new push-compatible options are added. Junio C Hamano approved the design direction while suggesting potential future refinement to handle all dash-prefixed arguments more uniformly. The series is now merge-ready with all technical concerns addressed.

### Fetch/push controls near completion

Derrick Stolee responded to Junio C Hamano's review comments on the negotiation control series, confirming two planned changes: updating the `--negotiate-only` error message to use `%s` placeholders for better translation safety, and switching to `OPT_ALIAS()` for the `--negotiation-tip` option to avoid string list duplication. The exchange demonstrates the series converging toward production readiness through final polish items, with no substantive changes to the underlying negotiation control functionality.

## In brief

**Pseudo-merge bitmap test infrastructure** -- Elijah Newren reviewed Taylor Blau's test helper implementation, pointing out a type mismatch in `bitmap_writer_push_commit()` and identifying documentation inconsistencies for the `sampleRate` parameter.

**Test suite `set -e` behavior** -- Jeff King confirmed brian m. carlson's findings about mksh's incorrect `set -e` behavior within `eval` statements, providing a minimal reproduction case that shows mksh diverging from POSIX requirements.

**UTF-8 truncation bugfix** -- Lorenzo Pegorari questioned whether the test case for control characters actually demonstrates the out-of-bounds read issue it was designed to catch, suggesting the test coverage could be strengthened.

**Tilde expansion in test scripts** -- Elijah Newren acked SZEDER Gábor's patch fixing tilde expansion issues in object filter tests, noting he couldn't find other instances of unquoted tildes in the test suite.

**`--reverse=before` revision walk** -- Ben Knoble and Mirko Faina engaged in detailed discussion about edge cases and implementation details for the new revision walking behavior, focusing on backward compatibility, error state handling, and `max_count` decrementing.

**`--missing=print-only` for rev-list** -- Derrick Stolee reviewed a new mode for partial clone workflows, raising concerns about implementation cleanliness and documentation while supporting the feature's purpose.

**Localization updates** -- Jiang Xin coordinated version 2 of the Git 2.54.0 l10n updates, focusing on Bulgarian and Traditional Chinese translations while including documentation additions for translation contributors.

## On the radar

**Source tree reorganization** -- Phillip Wood offered a nuanced review of Patrick Steinhardt's proposal to move libgit.a sources into a dedicated "lib/" directory, supporting the structural benefits while questioning whether it actually improves code navigation as claimed.

**Partial clone prefetching** -- Phillip Wood raised the possibility of generalizing `git cherry`'s prefetch logic to other commands like `git rebase`, identifying an opportunity to share optimization patterns more widely.