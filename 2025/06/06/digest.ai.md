# Git Mailing List Digest - 2025/06/06

**The day in brief.** A moderately busy day with 66 emails across 19 threads, featuring significant progress on several fronts. Key developments include the finalization of OAuth2.0 support in `imap-send`, resolution of a critical build system regression, and ongoing discussions about Change-ID standardization and submodule remote handling. The day also saw multiple bugfix series reaching maturity, with several likely to land in the next release.

## Notable threads

### OAuth2.0 support for imap-send reaches v14

Aditya Garg's long-running series to restore broken `imap-send` functionality and add OAuth2.0 support has reached its 14th iteration, with the latest version addressing final design questions about authentication method handling. The series fixes a critical configuration parsing bug that made `imap-send` unusable since Git 2.46.0 and implements RFC-compliant OAUTHBEARER and XOAUTH2 authentication methods. 

The v14 version introduces a `try_auth_method()` helper to reduce code duplication after Junio Hamano strongly advocated against the repetitive authentication blocks in earlier versions. The thread included extensive discussion about proper handling of `CURLOPT_PASSWORD` versus `CURLOPT_XOAUTH2_BEARER` and the various tradeoffs between strict failure and graceful fallback for unsupported methods. With all major technical issues resolved and the code structure now cleaned up, this series appears ready for final merging.

### Critical build system regression fixed

Johannes Sixt provided the definitive fix for a critical regression where `generate-tclindex.sh` was incorrectly deleting source TCL files during NO_TCLTK builds. The root cause was a misinterpretation of `$@` in the shell script context introduced in commit 2cc5b0facfa4. Randall S. Becker confirmed successful builds on NonStop systems after applying the fix, which changes the error cleanup path to explicitly target `lib/tclIndex` rather than using `$@`. 

This resolves an issue that had made git-gui unusable in NO_TCLTK configurations since Git 2.46.0. The thread included valuable discussion about build system practices, with Todd Zullinger detailing Fedora's packaging approach using config.mak for consistent make invocations. The fix is minimal and targeted, ensuring it can be safely included in the upcoming Git 2.50.0 release.

### Change-ID standardization discussions continue

The ongoing discussion about standardizing Change-IDs in commit footers saw substantive technical exchanges about determinism and uniqueness in distributed workflows. Toon Claes and Junio Hamano explored the tension between deterministic generation (valuable for tool interoperability) and the impossibility of enforcing global uniqueness in Git's distributed model. 

Key points included proposals for client-side validation of duplicate Change-IDs in reachable commits and the implications for different forge workflows (Gerrit vs PR-style). The thread also confirmed agreement on adding a config option to disable Change-ID generation for privacy-conscious users. While no final decisions were made, the discussion is converging on practical solutions that balance metadata utility with Git's decentralized nature.

### Batched reference updates refined

Karthik Nayak's series addressing edge cases in batched reference updates reached v3, with fixes for production issues encountered in GitLab's Gitaly service. The patches address a segfault in the files backend during failed updates and restructure `git receive-pack` to handle reference deletions separately from other updates, avoiding filesystem directory/file conflicts. 

The implementation now uses an enum for clearer phase separation and makes transaction creation lazy (only when updates exist), addressing feedback from earlier versions. With thorough test coverage in `t/t1400-update-ref.sh` and `t/t5516-fetch-push.sh`, and all major review feedback incorporated, this series appears ready for integration.

### Submodule remote handling challenges

Jacob Keller and Junio Hamano discussed implementation challenges in making submodule code properly handle non-standard remote names, particularly around deprecated functions in `remote.c` that incorrectly assume `the_repository`. The thread revealed deeper architectural questions about parameter passing in remote handling code as the project moves toward removing `the_repository` global.

Junio strongly endorsed modifying `read_remotes_file` and `read_branches_file` to properly accept repository pointers before their planned removal in Git 3.0, noting this would serve both immediate submodule needs and the broader architectural evolution. The discussion highlights how localized fixes can intersect with large-scale refactoring efforts like `the_repository` removal.

## In brief

**Stash option parsing fixes** -- Phillip Wood's series fixing `git stash -p` behavior and allowing more flexible option ordering has received positive reviews from Martin Ågren, who suggested minor test enhancements to better verify the interactive patch selection functionality.

**cURL type safety fixes completed** -- Johannes Schindelin and Jeff King's coordinated effort to address type mismatch warnings in cURL` calls reached conclusion, systematically fixing all 23 `curl_easy_setopt()` calls across the codebase to properly handle cURL 8.14.0's stricter type checking.

**Memory leak fixes** -- Lidong Yan and Junio Hamano collaborated on test coverage for a merged fix addressing memory leaks in `prepare_show_merge()`, with Lidong proposing a more comprehensive test scenario using actual merge conflicts rather than artificial index manipulation.

**Byte-order handling refactor** -- Sebastian Siewior reorganized macros in `compat/bswap.h` to correctly handle big-endian systems, fixing test failures on s390x that occurred after commit 6547d1c9cb added built-in bswap support.

**Windows branch deletion bug** -- Ondra Medek resolved their reported issue as a configuration error (duplicate URL entries in `.git/config`) rather than a Git bug, after Kristoffer Haugsbakk confirmed the problem didn't reproduce on Linux.

**Documentation rendering fix** -- A minor patch corrected Asciidoctor rendering in git-diff manpages by removing spurious backticks around ".." in a mode example, addressing malformed output in Asciidoctor 2.0.18.

## On the radar

**Stash import/export feature** -- Brian Carlson confirmed an upcoming v8 of their series adding `git stash` subcommands for importing and exporting stash entries via refs, which is expected to be the final iteration before potential inclusion.

**rebase-- prototype** -- An RFC was proposed for a new conflict resolution tool that leverages information from original merge commits during rebases, showing promise for certain scenarios but requiring significant work to integrate with Git's existing machinery.