# Git Mailing List Digest - 2025/06/01

## The day in brief
A moderately busy day with 37 emails across 9 threads, featuring significant progress on two major features: the long-awaited `git stash export/import` series reached its final form (v7) with all technical concerns resolved, and the `imap-send` improvements advanced to v11 with OAuth2.0 support and folder management now complete. Platform-specific fixes for BSD systems and documentation polish rounded out the day's activity.

## Notable threads

### Stash export/import reaches maturity
Patrick Steinhardt's `git stash export/import` series reached its seventh iteration with all major technical concerns resolved. The implementation now provides robust bidirectional transfer of stashes between repositories via refs under `refs/stash-export/`, preserving stash topology through commit chains. Key refinements in v7 include adopting Phillip Wood's `commit_list` optimization for topology validation and standardizing on existing reflog traversal APIs per Junio's guidance. The 4-patch series has consensus from all major reviewers and appears merge-ready after extensive review since 2022.

### IMAP-send modernization completes
Aditya Garg's comprehensive `imap-send` improvement series advanced to v11, addressing the final style concerns around error message formatting. The series makes `imap-send` usable again after a regression in Git 2.46.0 while adding significant new functionality: OAuth2.0 authentication support (both OAUTHBEARER and XOAUTH2 protocols), PLAIN authentication for OpenSSL, command-line folder specification via `--folder`, and a new `--list` option for displaying available IMAP folders. The implementation spans both OpenSSL and libcurl backends with thorough RFC compliance and memory management.

### Test path resolution terminology settled
The discussion around normalizing path handling in Git's test suite reached consensus on using "physical path" terminology (based on `pwd -P` behavior) to describe the resolution approach. Junio Hamano accepted Torsten Bögershausen's wording suggestion while noting Git's codebase more commonly uses "absolute" path terminology. The technical implementation (resolving paths with `cd -P` in test-lib.sh) was already settled; this finalizes the documentation language before merging.

## In brief

**NonStop reftable fix status** -- Randall S. Becker confirmed they're waiting for the approved `REFTABLE_UNUSED` macro fix to land in the upcoming rc1 release, as packagers can only build from official release candidates.

**BSD CPU detection improvement** -- Collin Funk confirmed Brad Smith's patch to use `HW_NCPUONLINE` instead of `HW_NCPU` on OpenBSD/NetBSD works correctly, ensuring accurate CPU count detection when SMT is disabled.

**BSD large memory detection** -- A new patch fixes physical memory detection on OpenBSD/NetBSD systems with ≥4GB RAM by adding support for the `HW_PHYSMEM64` sysctl while maintaining backward compatibility.

**git-column manpage fix** -- Version 2 of a documentation patch corrects erroneous list continuation markers in the `git-column` manpage examples that caused literal `+` characters to appear in rendered output.