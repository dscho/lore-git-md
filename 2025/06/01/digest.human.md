# Git Mailing List Digest - 2025/06/01

## The day in brief  
A moderately busy Sunday with 37 emails across 9 threads, dominated by two major patch series reaching maturity: the long-running `imap-send` improvements (now at v11) and the stash import/export functionality (v7). Both appear ready for merging after extensive review. Platform-specific fixes for BSD systems and documentation polish rounded out the day's traffic.

## Notable threads

### Stash import/export reaches final form  
Patrick Steinhardt's 4-patch series introducing `git stash export`/`import` subcommands has reached its seventh iteration with all technical concerns resolved. The implementation now provides robust bidirectional transfer of stashes between repositories via refs under `refs/stash-export/`, preserving topology through commit chains. Key refinements include adopting Phillip Wood's `commit_list` optimization, standardizing on reflog traversal APIs per Junio's guidance, and addressing platform-specific concerns. With 268 lines of coverage and consensus from all major reviewers, this represents the culmination of discussions since 2022 about stash portability.

### imap-send modernization completes  
Aditya Garg's comprehensive `imap-send` overhaul has progressed to v11, fixing a critical regression since Git 2.46.0 while adding OAuth2.0 support and folder management. The series now includes:
- Fixed configuration parsing that NULLed the folder field
- OAuth2.0 via OAUTHBEARER/XOAUTH2 for OpenSSL and libcurl
- PLAIN authentication for OpenSSL
- Memory leak fixes in CRAM-MD5
- New `--folder` command-line option
- RFC6154-compliant folder listing via `--list`

After extensive review of error message formatting to match Git conventions, the series appears technically complete with comprehensive authentication support and usability improvements.

## In brief  

**Meson test fix** -- Patrick Steinhardt addresses an unexpectedly passing macOS test in the Meson integration series, with Kristoffer Haugsbakk providing minor commit message polish.

**NonStop reftable fix pending** -- Randall S. Becker confirms awaiting merge of Carlo Marcelo Arenas Belón's `REFTABLE_UNUSED` macro fix for the upcoming rc1 release.

**BSD platform improvements** -- Two separate patches from Brad Smith improve CPU and memory detection on OpenBSD/NetBSD systems, correctly handling SMT-disabled configurations and >=4GB memory reporting.

**Manpage formatting fix** -- A v2 patch corrects erroneous `+` markers in `git-column` examples that appeared in rendered output, with Jean-Noël Avila acking the documentation change.

**Test path terminology settled** -- Junio Hamano and Torsten Bögershausen finalize "physical path" wording for test infrastructure changes normalizing symlink handling, noting the terminology aligns with `pwd -P` behavior.