# Git Mailing List Digest - 2025/10/17

**The day in brief.** A busy Friday with 76 emails across 22 threads saw significant progress on several fronts: Adrian Ratiu's hook subsystem refactoring reached v2 with comprehensive changes, Taylor Blau's massive repack refactoring series concluded review, and multiple documentation improvements landed. Security concerns emerged around SSH agent test handling, while a `git diff --quiet` regression prompted urgent discussion. The Git for Windows team also issued a critical security release addressing CVE-2025-26625.

## Notable threads

**Hook subsystem refactoring reaches maturity** -- Adrian Ratiu and Emily Shaffer's 10-part series to modernize Git's hook infrastructure reached v2 with significant architectural improvements. The series introduces callback-based stdin handling for parallel processes, output capture capabilities, and batched processing while maintaining backward compatibility. Key changes in v2 include stricter BUG() enforcement for mutually exclusive stdin methods, simplified post-rewrite and pre-push callbacks with strbuf reuse, and removal of problematic keepalive logic in update hooks. The implementation spans `run-command.c` parallel process handling, `hook.c` API extensions, and conversions in `receive-pack.c`, `sequencer.c`, and `transport.c`. Reviewers have largely approved the approach, with remaining feedback addressed through callback simplifications and documentation improvements.

**Repack refactoring concludes review** -- Taylor Blau's 49-part series refactoring Git's repack machinery to support incremental MIDX-based repacking while removing global variables reached final approval. The thread shows full consensus among reviewers (Jeff King, Patrick Steinhardt, and Junio Hamano) that the series is ready for merging after addressing v1 feedback around const-correctness, bool return types, and header cleanup. The changes successfully modularize the repack machinery while preserving legacy behavior, setting the stage for future MIDX/bitmap work. Reviewers noted several follow-up opportunities (marked as #leftoverbits) but considered these future work rather than blockers for the current series.

**SSH agent test handling reveals security concerns** -- A test failure report concerning `t7528-signed-commit-ssh.sh` uncovered deeper issues in how the test suite handles SSH agent setup. Jeff King identified that spaces in test directory paths break the unquoted `eval` of `ssh-agent`'s output, creating a silent fallback to any existing SSH_AUTH_SOCK in the environment. Junio Hamano raised security concerns about this potentially allowing arbitrary code execution if test paths contain shell commands. Discussion converged on using `$TMPDIR` for socket paths to avoid length limits and space issues, with plans to properly quote the evaluation. This revealed both a test isolation problem and a security vulnerability in the test infrastructure.

**`git diff --quiet` regression sparks urgent fix** -- Jake Zimmerman reported and Jeff King confirmed a serious regression in Git v2.51.1 where `git diff --quiet HEAD` incorrectly produces output for staged files. The bisect pointed to commit b55e6d36eb ("diff: ensure consistent diff behavior with ignore options") as the culprit. Jeff proposed a conservative fix redirecting output to /dev/null when `diff_from_contents` is active in quiet mode, which Junio strongly endorsed as a "stupid but cannot be incorrect" approach. The discussion revealed additional unguarded output paths in the diff machinery that may need addressing in follow-up work.

**Git for Windows security release** -- Johannes Schindelin announced Git for Windows 2.51.1 addressing high-severity CVE-2025-26625 in Git LFS. The urgent release also updates bundled components (OpenSSL 3.5.4, OpenSSH 10.2.P1) and removes deprecated tools. Notably, the announcement states `git svn` support will be dropped in the next major release due to maintenance challenges. Windows users are strongly advised to upgrade immediately given the security implications.

## In brief

**Promisor pack object handling bug** -- Sam Estep provided a reliable reproduction case for intermittent BUG() assertions in `should_include_obj()` during promisor pack operations, moving the thread closer to a fix.

**Documentation formatting fixes** -- Kristoffer Haugsbakk and Julia Evans collaborated on post-merge polish for `git-checkout` man page formatting, correcting AsciiDoc markup inconsistencies.

**gitk window geometry persistence** -- Johannes Sixt finalized a patch restoring full window geometry saving in gitk, with Mark Levedahl confirming the historical context is now properly documented.

**Technical documentation standardization** -- Ramsay Jones completed a series standardizing technical docs to asciidoc format with the addition of `large-object-promisors.adoc` to build systems.

**CR/LF handling in GPG verification** -- Christian Couder provided detailed feedback on Okhuomon Ajayi's patch to only remove CR before LF in signature verification, focusing on commit message improvements.

**On the radar**

**Commit-graph segfault with alternates** -- Philipp Hahn reported a segmentation fault when commit-graph files in alternate repositories become inaccessible, suggesting either automatic rebuilds or user warnings as solutions.

**`git last-modified` performance work** -- Toon Claes' bitmap-based optimization for path modification tracking continues refinement, with recent discussion focusing on memory management and flag usage clarity.