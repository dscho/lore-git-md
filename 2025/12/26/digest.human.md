# Git Mailing List Digest - 2025/12/26

**The day in brief.** A moderately active day with 26 emails across 9 threads, featuring the completion of a major hook subsystem refactoring series, several bugfix discussions, and performance optimizations. The standout development is the v6 submission of Adrian Ratiu's hook API standardization work, marking the culmination of a multi-year effort to modernize Git's hook infrastructure.

## Notable threads

### Hook subsystem refactoring reaches completion

Adrian Ratiu submitted the final version (v6) of his 11-part series refactoring Git's hook subsystem, representing the completion of a multi-year effort to standardize hook implementation on a new `hook.h` API. The series converts all major hooks to use structured APIs for stdin handling, output capture, and parallel execution while maintaining backward compatibility. Key improvements include batched stdin processing (500 lines per callback) for receive hooks, sideband output preservation, and removal of direct `find_hook()` calls. With all major review feedback addressed and sign-offs from key reviewers (Ævar Arnfjörð Bjarmason and Emily Shaffer), this foundational work is now ready for merging in an upcoming release cycle.

### `git branch --show-current` flag behavior resolved

A discussion about `git branch --show-current`'s silent ignoring of the `-v` flag concluded with a decision to make the combination error out explicitly. Junio Hamano initially noted the current behavior was intentional (as the command was designed as a simple alternative to `git symbolic-ref HEAD`) but later agreed the silent flag dropping was buggy. Sergey Organov submitted a patch implementing the error check, which Junio accepted as the simpler solution compared to implementing verbose output support. The change maintains backward compatibility while fixing what was previously considered buggy behavior.

### Batched reference update error reporting discussion

Karthik Nayak responded to Jeff King's proposed fix for restoring detailed error messages lost in Git 2.51.0 when pushing non-commit objects. The discussion converged on attaching error details to individual failed ref updates rather than the whole transaction, with Karthik confirming the technical approach and noting he'll implement the changes after the New Year's break. The solution will modify `ref_transaction_maybe_set_rejected()` to accept error details and store them in a new `rejection_details` field, addressing a regression affecting push, fetch, and update-ref operations across multiple ref backends.

### `pack-refs` performance investigation

Jeff King and Brian m. carlson continued their investigation into `git pack-refs` performance, particularly why it's slower than JGit for repositories with high ref counts. The discussion revealed that Git's need to peel tags by reading object headers creates significant overhead, even for refs that theoretically shouldn't need peeling (like those under `refs/heads/`). Brian noted the reftable backend might avoid this overhead entirely by maintaining pre-peeled references, suggesting potential long-term performance advantages for high-refcount scenarios.

## In brief

**`show-branch` performance optimization** -- René Scharfe replaced `commit_list` with a `prio_queue` in `builtin/show-branch.c`, improving worst-case complexity from O(n²) to O(n log n) with a 98.6% speedup in pathological cases.

**Documentation feedback for promisor remotes** -- Jean-Noël Avila suggested improvements to Christian Couder's documentation patch for the LOP (Large Object Promisors) series, noting formatting and clarity issues in the `--filter` option description.

**`git subtree` regression report** -- A user reported a regression in subtree push functionality when using `--squash`, tracing it to commit 83f9dad7d6 which appears to break history reconstruction for squashed commits.

**Maintainer feature proposal volunteer** -- Usman Akinyemi volunteered to implement Junio Hamano's proposed push group functionality enhancement, which would add symmetry between fetch and push operations when using remote groups in Git config.

## On the radar

**Configurable branch comparison in `git status`** -- Harald Nordgren's series enabling `status.goalBranch` configuration appears complete after 8 iterations, with only minor procedural questions about commit organization remaining before potential merge.