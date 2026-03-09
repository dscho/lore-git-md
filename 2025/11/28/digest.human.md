# Git Mailing List Digest — 2025/11/28

**The day in brief.** A moderately active Friday with 28 emails across 11 threads, featuring important fixes for the new `git last-modified` command, detailed design discussions around `git replay --revert` behavior, and several documentation improvements. The most critical developments were cross-platform bug fixes in the recently merged `last-modified` optimization and substantive progress on the `replay` feature's edge case handling.

## Notable threads

### `git last-modified` cross-platform fixes

Toon Claes addressed critical issues in the recently merged `git last-modified` optimization that were causing test failures on s390x and NonStop systems. The bug, initially thought to be big-endian specific, was actually an algorithmic problem in merge commit handling that affected all platforms. The fix corrects how merge commit parent ordering is processed in the generation-sorted priority queue while preserving the command's 2.5x-111.7x performance gains. Jeff King and Anders Kaseorg contributed to diagnosing the issue, which manifested when files were incorrectly attributed to merge commits rather than their actual sources. A separate follow-up patch fixed bitmap initialization that was causing memory corruption, particularly visible on s390x.

### `git replay --revert` design refinements

The discussion around `git replay --revert` reached important clarity about its handling of disconnected commits. Elijah Newren and Junio Hamano revealed that the current implementation doesn't actually support reverting non-contiguous commits as initially thought — it follows standard revision range behavior by processing all ancestors of listed commits. True disconnected commit support would require implementing `--no-walk` functionality, which currently fails with errors or segfaults. The thread converged on focusing the current series on basic `--revert` functionality while outlining a more sophisticated future approach for disconnected commit handling that would process commit ranges and individual commits as independent "islands."

### Dropping empty commits in `git replay`

Phillip Wood's patch to make `git replay` drop commits that become empty during replay (matching `rebase` and `cherry-pick` behavior) received positive feedback from both Junio Hamano and Elijah Newren. The change addresses cases where keeping such commits would leave misleading commit messages. Elijah suggested test simplifications to verify the behavior more clearly, including using `--contained` instead of complex ancestry-path queries. The discussion resolved an open question about `result->tree` assignment by approving Phillip's change to use a temporary variable.

## In brief

**German translation fix** — Sören Krecker confirmed "Revision" is the correct German term for `git rev-parse --verify` error messages, addressing brian m. carlson's report about the current inaccurate "Commit" translation.

**Zsh test compatibility** — Two minimal patches fixed test suite issues when run under zsh's POSIX-compatible mode, addressing a hanging SOCKS server test and reftable line count comparison.

**Reference backend URI proposal** — A new `GIT_REF_URI` environment variable was proposed to specify reference backend types and locations using URI syntax (e.g., `files:/path/to/refs`).

**Documentation improvements** — Kristoffer Haugsbakk updated advice messages to prefer `git help` over `man` for manual page references, while Stefan Rieche fixed a command syntax typo in git-pull's man page.

**`git fetch` SIGSEGV report** — Justin Su reported a segmentation fault during submodule operations on ARM64, though brian m. carlson's analysis suggested repository corruption might be the root cause.