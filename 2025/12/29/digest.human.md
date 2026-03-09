# Git Mailing List Digest — 2025/12/29

## The day in brief

A moderately active day with 10 emails across 9 threads, featuring Junio's "What's cooking" report, several performance optimizations and bugfixes, and a new contributor introduction. The standout items are René Scharfe's `show-branch` optimization (validated with 23x speedup in real-world use) and a race condition fix for `git fsck` in live repositories.

## Notable threads

### "What's cooking" report

Junio C Hamano's regular status update outlines the current state of Git development branches. Twelve topics have graduated to 'master', including promisor object optimizations and documentation updates. Eight new proposals are under consideration, ranging from "git replay" error handling to a major refactoring using the commit_stack API. Thirty ongoing topics are in various stages of review, covering Windows symlink support, MIDX improvements, and the new "git history" command. One proposal (single-letter option completion) was rejected outright. The report highlights active development across performance, Windows support, documentation, and API cleanups.

### `show-branch` performance optimization

René Scharfe's patch to replace `commit_list` with `prio_queue` in `show-branch` received strong validation from Derrick Stolee, who confirmed a 23x speedup in his monorepo tests. The optimization avoids O(n²) behavior by using a heap-based queue, with Stolee particularly noting the effective use of `prio_queue_replace()` to avoid double-sift operations. The exchange demonstrates how synthetic test cases (which showed a 46x improvement) can meaningfully predict real-world gains, albeit with some variance. The thread shows clear technical consensus on this being a worthwhile change.

### `fsck` race condition fix

A significant bugfix addresses a race condition in `git fsck` when run on live repositories, where concurrent modifications could cause false positives about missing commits. The solution introduces a snapshot mechanism that captures ref states at operation start, with co-authorship from Elijah Newren helping adapt it for upstream. The patch handles both refs and reflogs, making snapshotting the default behavior when no specific refs are provided. A new test case demonstrates the fix working against concurrent commit operations. This change will particularly benefit forge operators and automated maintenance jobs where repository modifications during `fsck` are common.

## In brief

**Merge-ort submodule conflict fix** -- Addresses an assertion failure when merging branches that treat a path as both a directory and submodule, particularly in cases with multiple merge bases.

**Test modernization** -- A GSoC participant contributes a shell `test -f` replacement with `test_path_is_file` in the post-checkout hook test, improving failure diagnostics.

**Diff machinery race condition** -- Fixes a segmentation fault in partial clones when diff queue entries are freed while parallel prefetch operations access them.

**New contributor introduction** -- Samuel Abraham Adekunle introduces himself as a prospective GSoC 2026 participant interested in low-level systems work.

**Windows deployment inquiry** -- A contributor directs a question about MSI packages to Git for Windows, clarifying Git's source-only distribution model.