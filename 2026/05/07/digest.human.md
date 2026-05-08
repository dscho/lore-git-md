# Git Mailing List Digest - 2026/05/07 (Thursday)

**The day in brief.** A moderately busy Thursday with 52 emails across 19 threads, featuring the completion of a major autostash feature, a new `git format-rev` command proposal, and several platform-specific fixes. The most notable developments include the final administrative check-in for the 21-iteration `checkout -m` autostash series and Kristoffer Haugsbakk's well-reviewed `format-rev` command reaching v4.

## Notable threads

### Autostash for `checkout -m` concludes

Harald Nordgren's long-running effort to implement autostash behavior for `git checkout -m` has reached its final administrative check-in after 21 iterations (14 pre-merge and 7 post-merge). The feature provides comprehensive conflict resolution customization during branch switching operations, including stash message customization and lockfile cleanup. Reviewers Phillip Wood and Junio C Hamano have provided their final sign-offs, with only two non-blocking suggestions remaining for potential future polish. This represents the completion of a significant feature that unifies conflict resolution between autostash and manual stash operations.

### `git format-rev` command proposal

Kristoffer Haugsbawk's v4 series introduces a new `git format-rev` command for on-demand pretty formatting of revisions from stdin. The implementation provides two input modes (`--stdin-mode=revs` for commit-per-line and `--stdin-mode=text` for freeform text parsing) along with NUL-termination support and robust stream handling. The series demonstrates excellent patch hygiene with clear separation of mechanical changes from functional work, comprehensive test coverage, and responsiveness to technical feedback from reviewers Phillip Wood and Ramsay Jones. Marked as EXPERIMENTAL to allow for future refinement based on real-world usage.

### Windows geometric repack deadlock fix

Johannes Schindelin's two-patch series addresses a Windows-specific deadlock in geometric repacking during `git maintenance`. The root cause was missing `child.odb_to_close` settings when spawning `git repack`, leaving mmap'd files locked on pre-POSIX-delete Windows versions. The fix includes test infrastructure to simulate legacy Windows delete behavior and has received positive reviews from Derrick Stolee and Patrick Steinhardt. This represents a well-contained platform-specific fix with thoughtful test coverage.

### Dash shell multibyte character fix

Herbert Xu, maintainer of the Dash shell, provides an upstream fix for multibyte character handling in heredocs that was causing test failures in Git's test suite. This completes the cross-project collaboration initiated by Patrick Steinhardt's report, addressing the root cause rather than just working around the issue in Git's tests. The fix modifies Dash's parser to properly process multibyte characters in quoted heredoc contexts, preventing incorrect escaping that was corrupting test inputs.

## In brief

**`git add -p` word-diff view** -- Javier Bassi proposes adding a 'w' command to show hunks with word-diff formatting, receiving substantive technical review from Phillip Wood about filename propagation and context preservation needs.

**Documenting `--no-follow`** -- Tamir Duberstein's v2 patch refines documentation for the long-standing but previously undocumented `--no-follow` option in `git log`, improving precision about pathspec handling.

**Nedmalloc removal** -- Johannes Schindelin completes removal of the unmaintained nedmalloc allocator from Windows builds in a six-patch series addressing GCC 16 compatibility issues.

**RHEL 6 build workaround** -- A user confirms resolving OpenSSL version conflicts by explicitly pointing to a newer OpenSSL installation, documenting a solution for unsupported platforms.

**Commit-graph documentation** -- Kristoffer Haugsbawk's v3 patch documenting commit-graph behavior with replace objects receives final Acked-by from Derrick Stolee after refining performance impact wording.

## On the radar

**Branch pruning design** -- Harald Nordgren's local branch pruning feature faces architectural reconsideration after Johannes Sixt's objection to integrating it with `git fetch`, despite successful real-world validation.

**Merge support in `git history`** -- The RFC series adding merge commit support continues receiving technical feedback, with discussions about error handling edge cases and patch organization.