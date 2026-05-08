Here's the daily digest for 2026/05/07:

---

### The day in brief
A moderately busy Thursday with 52 emails across 19 threads, featuring several notable technical discussions and patch series nearing completion. Key highlights include the conclusion of a 21-iteration autostash feature, a new `git format-rev` command proposal, and substantive reviews on multiple fronts. The day saw productive cross-platform collaboration on Windows-specific fixes and continued refinement of documentation.

---

### Notable threads

**Autostash for `checkout -m` reaches completion**  
Harald Nordgren's long-running effort to implement autostash behavior for `git checkout -m` has concluded after 21 iterations (14 pre-merge and 7 post-merge). The feature provides comprehensive conflict resolution customization during branch switching, including stash message customization and lockfile cleanup. Reviewers Phillip Wood and Junio C Hamano have provided final sign-offs, with only two non-blocking suggestions remaining. This represents a significant milestone in Git's branch switching capabilities, demonstrating the project's rigorous review process for complex features.

**New `git format-rev` command proposed**  
Kristoffer Haugsbakk's v4 series introduces an experimental `git format-rev` command for on-demand pretty formatting of revisions from stdin. The implementation includes two input modes (line-based revisions and freeform text), NUL-termination support, and robust stream handling. The series has evolved through substantive technical review, with Phillip Wood, Ramsay Jones, and Junio Hamano providing feedback on control flow and safety characteristics. Marked as EXPERIMENTAL to allow for future refinement, this fills a gap in Git's revision formatting capabilities for stdin processing scenarios.

**Windows geometric repack deadlock fix**  
Johannes Schindelin's two-patch series addresses a Windows-specific deadlock in geometric repacking during maintenance operations. The root cause was mmap'd pack index files preventing deletion on pre-POSIX-delete Windows versions. The fix properly sets `odb_to_close` when spawning `git repack`, matching other maintenance tasks' behavior. The series includes test infrastructure to simulate legacy Windows delete semantics and has received positive reviews from Derrick Stolee and Patrick Steinhardt. This demonstrates Git's cross-platform maintenance challenges and the project's systematic approach to Windows-specific issues.

**`git add -p` word-diff enhancement review**  
Javier Bassi's proposal to add word-diff viewing to `git add -p` received substantive technical reviews from Phillip Wood and D. Ben Knoble. The discussion covers marker parsing edge cases, Windows CRLF handling, and interface design considerations. Knoble suggested a more extensible `:show <opts>` command syntax inspired by Vim, while Wood focused on technical requirements like filename propagation and context preservation. The reviews demonstrate Git's attention to detail when adding new interactive features, particularly around platform compatibility and user interface design.

**In brief**  
**Dash shell heredoc fix** -- Herbert Xu provides an upstream fix for multibyte character handling in heredocs that was causing Git test failures, completing cross-project collaboration.

**RHEL 6 build workaround confirmed** -- Original reporter documents successful resolution of OpenSSL version conflict using explicit `--with-openssl` path.

**Documentation refinements** -- Tamir Duberstein's v2 patch improves precision in documenting `git log --no-follow` option's pathspec handling.

**Nedmalloc removal completed** -- Johannes Schindelin's series fully excises the unmaintained allocator from Windows builds in six methodical patches.

**Rebase --update-refs test improvements** -- Phillip Wood suggests more robust test verification for a bugfix preventing invalid ref updates.

**On the radar**  
**Branch pruning design crossroads** -- Harald Nordgren's local branch pruning feature faces architectural reconsideration after Johannes Sixt's objection to fetch integration.

**Merge support in git history** -- RFC series adding merge commit handling continues to receive detailed feedback about error case handling.

**Build system warning suppression** -- Parallel discussions emerge about handling Clang's C11 extension warnings with glibc headers.

--- 

Today's activity reflects Git's ongoing balance between new feature development, platform-specific maintenance, and documentation improvement. Several long-running efforts reached significant milestones while newer proposals received thorough technical vetting from domain experts. The collaborative review culture remains strong, with particular attention to edge cases and platform compatibility concerns.