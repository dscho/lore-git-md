# Git Mailing List Digest — 2025/03/28

**The day in brief.** A moderately busy Friday with 45 emails across 19 threads saw several long-running efforts reach important milestones. The reftable decoupling series cleared its final Windows-specific blocker, Karthik Nayak's batched reference updates received final approval, and the Perl removal from tests was reviewed positively. Meanwhile, platform-specific issues dominated troubleshooting efforts, particularly around s390x test failures and Cygwin regressions.

## Notable threads

### Batched reference updates ready for merge

Karthik Nayak's series introducing partial failure support for batched reference updates received its final review approvals today. Patrick Steinhardt confirmed the v5 changes look good, noting stylistic improvements and optimizations like the `conflicting_dirnames` strset optimization. Jean-Noël Avila followed up with a minor documentation formatting fix for the manpage synopsis, clarifying how alternative command invocations should be presented. With all technical concerns addressed across five iterations, this major refs infrastructure change appears poised for merging in the next cycle.

### Windows lockfile race condition resolution

After extensive debate about Windows-specific race conditions in lockfile creation, the thread converged on using the `RtlGetLastNtStatus()` API despite initial objections about its undocumented status. Johannes Schindelin reluctantly accepted this solution as the only approach that precisely detects pending deletions, addressing his own concerns about initialization timing. The discussion revealed the technical tradeoffs between correctness (NtStatus) and maintainability (SQLite-style polling), with Windows-specific considerations about API stability ultimately favoring the more precise solution.

### Reftable decoupling clears Windows hurdle

Patrick Steinhardt confirmed the reftable decoupling series is now ready to merge after resolving the final MinGW-specific memory allocator issue. The fix involved moving mimalloc override declarations to `compat/posix.h`, verified through successful CI runs on Git for Windows. Johannes Schindelin confirmed the solution works in Git for Windows' `shears/seen` branch, noting this marks the first successful CI build in weeks. This clears the path for a significant step in the long-term ref backend abstraction work by removing Git-specific dependencies from reftable code.

### Bash function detection improvements

A third iteration of improvements to Bash function detection in Git's diff machinery arrived today, addressing limitations in the existing userdiff driver's pattern matching. The patch simplifies function detection by capturing entire definition lines rather than trying to match specific body patterns, while expanding word regex coverage for Bash syntax elements like parameter expansion operators. The implementation includes comprehensive new test coverage and represents a synthesis of feedback from shell expert Johannes Sixt and maintainer Junio Hamano, balancing improved functionality with maintainability concerns.

## In brief

The final version of Patrick Steinhardt's Perl removal series received positive review from Phillip Wood, who noted the removal of "useless indirections for sed(1)" as an improvement. Karthik Nayak's `git blame` porcelain output markers patch received minor code review feedback from Patrick Steinhardt, suggesting potential function name improvements and `puts()` usage. The GSoC 2025 git-refs consolidation project saw technical clarifications from Shejialuo about project scope and test strategy. A build system fixes series collected various small improvements including meson curl detection fixes and gitweb.js generation corrections. Documentation fixes addressed conflict markers in AsciiDoc files and improved `git version --build-options` SHA implementation reporting. A rebase status reporting series improved merge commit handling and status output formatting during interactive rebase. Performance test fixes corrected prerequisite handling in grep engine tests and scalar clone behavior in detached HEAD state.

## On the radar

The `backfill --min-batch-size` test failures on s390x and sparc64 architectures remain under investigation, with Patrick Steinhardt providing concrete debugging instructions to gather more diagnostic information. A Cygwin regression in gitk causing stack exhaustion is being examined, with Johannes Sixt questioning whether Windows-specific security measures should apply to Cygwin. The `git push --mirror` hang with Azure DevOps was reported, with initial analysis suggesting server-specific or TLS-related issues rather than a Git bug.