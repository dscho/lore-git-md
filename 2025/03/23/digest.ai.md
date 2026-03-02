# Git Mailing List Digest — 2025/03/23

**The day in brief.** A moderately active Sunday with 24 emails across 15 threads, featuring security hardening for git-shell command overrides, final polish on several patch series nearing integration, and multiple GSoC project inquiries. The most active discussions centered on the git-shell security model and `git blame` porcelain output compatibility.

## Notable threads

### git-shell command override security model finalized

Ayman Bagabas's series allowing git-shell built-in commands to be overridden by scripts in git-shell-commands reached its final form after extensive security review. The v3 patch ([2025/03/23/15-29-30]) incorporates feedback to use `X_OK` rather than `F_OK` for permission checks, addressing TOCTOU race condition concerns while maintaining git-shell's existing security model. Chris Torek provided key implementation guidance, noting that while `access()` has inherent race conditions, the approach aligns with git-shell's current trust boundaries. The series appears ready for maintainer consideration after this methodical progression from proposal through security hardening.

### `git blame` porcelain format compatibility concerns

Junio Hamano raised important design considerations ([2025/03/23/15-58-03]) about extending `git blame` porcelain output to show ignored/unblamable line markers. While agreeing the markers should be visible, he cautioned that prefixing them directly to commit hash lines would break existing parsers expecting strict 40-character SHA-1 formats. The discussion shifted to interface design, with Hamano suggesting additional metadata lines rather than modifying record delimiters. This thread now needs an alternative proposal that maintains backward compatibility while exposing the new information.

### SMTP error handling faces test failures

Zheng Yuting's GSoC project to make `git-send-email` SMTP error handling RFC 5321 compliant hit a snag ([2025/03/23/02-21-09]), with the series failing 169/215 tests in t9001-send-email.sh. The technical approach — using regex-extracted status codes to distinguish temporary (4xx) from permanent (5xx) errors — appears sound, but the widespread test failures suggest either necessary test updates for the new RFC-compliant behavior or deeper compatibility issues. The series remains blocked pending investigation into whether these represent expected behavior changes or actual regressions.

## In brief

Justin Tobler's v2 series fixing default branch advice suppression during clone operations received positive review from Junio Hamano ([2025/03/23/19-38-03]), with approval of its flags-based approach and added test coverage. The documentation modernization of MyFirstContribution tutorial ([2025/03/23/22-08-06]) is down to final style nits about C++-style comments. David Mandelberg submitted final optimizations ([2025/03/23/21-05-46]) for the already-accepted completion fix for remote names with slashes, checking if minor updates to queued patches are still acceptable. A refactoring in commit.c ([2025/03/23/09-53-21]) simplified commit mark clearing functions by consolidating the single-commit handling logic.

## On the radar

The bulk-checkin sign comparison discussion ([2025/03/23/22-08-03]) concluded with Junio Hamano's clarification about type safety guarantees, providing useful context for future similar changes. Two GSoC project inquiries ([2025/03/23/13-36-51], [2025/03/23/14-50-18]) showed early interest in refs consolidation and repository information commands, though both proposals need further technical development. Jean-Noël Avila responded to a Greek translation inquiry ([2025/03/23/11-24-38]), potentially reviving the dormant progit2-gr effort.