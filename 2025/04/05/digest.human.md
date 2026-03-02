# Git Mailing List Digest — 2025/04/05

**The day in brief.** A moderately busy Saturday with 25 emails across 15 threads, highlighted by a major milestone in Git's merge machinery evolution as Elijah Newren's series to remove the legacy merge-recursive backend reached its final form. Other notable activity included GSoC proposal refinements for the `the_repository` removal effort and continued discussion around `git stash` usability improvements.

## Notable threads

### Merge-recursive removal completed

Elijah Newren's 8-part series to fully remove the legacy merge-recursive backend in favor of merge-ort reached its v2 iteration, representing the culmination of a multi-year architectural effort. The series systematically converts all remaining callers (in checkout, merge, rebase, and revert commands) to use merge-ort exclusively before deleting merge-recursive.[ch] and its test infrastructure. The changes remove over 5000 lines of code while maintaining all functionality, as merge-ort has been Git's default strategy since 2.33.0. The v2 changes are purely editorial improvements to commit messages, indicating the technical content is stable and ready for merging.

### GSoC proposals for `the_repository` removal

Two GSoC applicants refined their proposals for the ongoing effort to reduce Git's global state. Arnav Bhate presented a structured 10-week plan focusing first on environment.c's globals before tackling repository.h components, while Anthony Wang proposed a more modest 90-hour effort targeting environment.c specifically. Both demonstrate growing familiarity with the codebase through prior contributions (Bhate's sign comparison fixes) or coursework (Wang's C experience). The proposals reflect careful consideration of Patrick Steinhardt's earlier feedback about breaking the work into measurable deliverables.

### `git stash list` metadata proposal

A feature proposal suggested enhancing `git stash list` output with metadata like line changes, affected files, and stash age to help users better understand WIP stashes without manual inspection. The suggested format would show: `stash@{10}: WIP on main: [commit-subject] (+20, -5, 3 dirs, 5 files, 17 days ago)`. Lucas Seiki Oshiro countered that existing `git log --stat` options could provide similar information, potentially making code changes unnecessary. The discussion remains open on whether to implement new functionality or better document current capabilities.

## In brief

Jeff King's `git fetch` remote HEAD tracking fix received final approval from original author Bence Ferdinandy, clearing the way for merging. Jean-Noël Avila's documentation modernization work prompted a follow-up about ellipsis formatting in `git-mv` synopses. A UTF-8 comparison fix for packed-refs gained additional filesystem compatibility validation from brian m. carlson. Christian Fredrik Johnsen submitted a v2 typo fix for a refs.c comment after addressing Martin Ågren's formatting feedback. Nikolay Shustov followed up on his git-p4 encoding fix, awaiting test results from Fahad Al-Rashed's environment.

## On the radar

The Change-ID discussion continues to explore practical usage patterns, with Theodore Ts'o sharing Linux kernel development experiences about their utility in Gerrit versus native Git workflows. A feature inquiry about multi-remote synchronization workflows may spark discussion about Git's mirroring capabilities, though no solutions have been proposed yet.