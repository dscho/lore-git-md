# Git Mailing List Digest - August 13, 2025

**The day in brief.** A busy Wednesday with 63 emails across 18 threads, featuring the second release candidate for Git 2.51.0, significant progress on reftable/libgit2 compatibility, and ongoing discussions about documentation improvements. Key highlights include Patrick Steinhardt's reftable series reaching v4 with all feedback addressed, and Julia Evans' documentation patches evolving through community feedback.

## Notable Threads

### Reftable/LibGit2 Compatibility Series Reaches v4

Patrick Steinhardt's series to make reftable production-ready for libgit2 integration has progressed to v4, addressing all technical feedback from reviewers. The 8-patch series standardizes error handling, implements atomic stack modifications, and fixes race conditions in compaction operations. Key changes in this iteration include documentation fixes and confirmation that libgit2's test suite now passes with reftable support (though Windows compatibility and memory leaks remain outstanding). The series has received positive reviews from Justin Tobler, Carlo Arenas, and Junio Hamano, with no remaining technical objections. This work is particularly significant as reftable is slated to become the default ref storage format in Git 3.0.

### Git 2.51.0-rc2 Released

Junio Hamano announced the second release candidate for Git 2.51.0, featuring 494 non-merge commits from 78 contributors (including 20 new contributors). Notable changes include UI improvements like R language userdiff patterns, performance enhancements in `git fetch`/`git push` through batched reference updates, and continued internal refactoring to remove `the_repository` usage. The release also declares the reftable backend mature enough to become the default format in Git 3.0. This RC appears to be the final one before the official release unless critical issues emerge.

### Documentation Improvements Debate

Julia Evans' documentation patch series for `git-add` has evolved to v2 after extensive discussion about balancing accessibility with technical completeness. The revised series drops controversial SYNOPSIS changes that drew mixed feedback, focusing instead on DESCRIPTION section improvements that received broader support. Changes include adding concrete examples-driven explanations of the index/staging area, simplifying ignored file documentation, and clarifying Git's terminology inconsistencies. The thread highlights ongoing tensions between making documentation approachable for beginners while maintaining completeness for experienced users, with Jean-Noël Avila advocating for comprehensive reference material while Julia emphasizes pedagogical clarity.

### `git stash --only-unstaged` Feature Proposal

A new feature proposal suggests adding `--only-unstaged` to `git stash push` to improve commit-splitting workflows during interactive rebases. The option would create stash entries containing only working tree changes, leaving the index untouched - addressing a pain point where current solutions like `--keep-index` can cause conflicts when popping stashes later. D. Ben Knoble provided detailed analysis showing how current stash internals lead to these conflicts, reinforcing the proposal's motivation. Junio Hamano is probing for potential edge cases and workflow differences from existing options, suggesting this may progress to implementation if the use case holds up under scrutiny.

### `git send-email` Documentation Clarification

A thread about `git send-email`'s `--suppress-cc` option behavior concluded with a documentation patch from Junio Hamano. The change makes explicit that the option can be given multiple times (once per category) to suppress different types of automatic CCs, rather than accepting comma-separated values as some users expected. This resolves confusion while requiring no code changes, simply clarifying the existing behavior in the documentation. The exchange demonstrates Git's careful documentation practices, where even long-standing behaviors may need explicit documentation when they diverge from user expectations.

## In Brief

**Test infrastructure modernization** -- D. Ben Knoble's v5 series modernizing editor-related tests in t7005-editor.sh incorporates performance optimizations from Phillip Wood, preferring direct `VAR=val cmd` syntax over `test_env` except for shell function calls.

**Commit-graph globals removal** -- Consensus forms to split Patrick Steinhardt's series, proceeding with `the_repository` removal from commit-graph code while deferring non-essential integer type changes for separate discussion.

**Rebase documentation internals** -- Technical discussion continues about revision range notation behavior, with Junio Hamano confirming that `git rev-list --count --left-only @{u}...` and `git rev-list --count ..@{u}` should produce equivalent results when not using `--cherry-pick`.

**Git for Windows 2.51.0-rc2** -- Johannes Schindelin announced the Windows port's second release candidate, featuring updates to 7-Zip, cURL, MSYS2 runtime, and MinTTY dependencies.

**Potential null pointer dereference** -- Phillip Wood questions whether a reported null pointer issue in `builtin/describe.c` is actually reachable in practice, as all callers appear to pass verified commit OIDs to `describe_commit()`.

**Diff output formatting** -- Maintainers defend the current "\ No newline" marker behavior as technically correct, showing each version's newline status independently rather than implying changes where none exist.

**Gitignore include proposal** -- Ryan Johnson suggests adding an `include` directive to `.gitignore` files, though brian m. carlson raises concerns about backward compatibility and ecosystem impact.