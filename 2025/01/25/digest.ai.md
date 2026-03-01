# Git Mailing List Digest — 2025/01/25

## The day in brief

A moderately active day with 10 emails across 5 threads, featuring a mix of platform-specific fixes, authentication improvements, and architectural discussions. The standout items are a new OAuth authentication option for `git-send-email` and an ongoing debate about Windows file handling in the reftable backend. A GitHub-focused TUI proposal generated brief but definitive feedback about Git's forge-agnostic philosophy.

## Notable threads

**OAuth support for git-send-email**  
A feature patch adds OAuth bearer authentication (OAUTHBEARER and XOAUTH2 mechanisms) to `git-send-email`, working around limitations of Perl's Authen::SASL module. The implementation includes new helper functions for generating authentication strings, proper TLS enforcement per RFCs 5801 and 7628, and documentation updates with example configurations for Gmail and Office 365. This v2 revision addresses formatting and minor code issues from the initial submission, with positive test feedback from the git-credential-oauth maintainer. The change enables integration with major email providers while requiring proper credential helper configuration for token management.

**Windows reftable file handling debate**  
An architectural discussion continues about how to handle Windows' non-POSIX file locking behavior in the reftable backend, where failed unlinks are expected but Windows' compatibility layer currently prompts users. Patrick Steinhardt's initial patch proposed disabling retry behavior specifically for reftable via a new `MINGW_DONT_HANDLE_IN_USE_ERROR` flag. Johannes Sixt questioned whether the solution belongs in the platform layer or reftable implementation, prompting Steinhardt to clarify JGit compatibility constraints. Christian Reich confirmed the patch's functionality but noted merge conflicts with Git for Windows, while Sixt later proposed alternative approaches using `CreateFileW` with `DELETE` access flags. The thread remains unresolved but has surfaced important considerations about subsystem boundaries and platform abstraction.

**GitPanel TUI proposal declined**  
A proposal for "GitPanel" — a terminal UI for GitHub account management — received clear feedback from brian m. carlson that such forge-specific functionality falls outside Git's scope. While acknowledging the concept's potential usefulness, the response emphasized Git's forge-agnostic philosophy and suggested developing the tool as a standalone project supporting multiple forges, similar to tig. The exchange demonstrates Git's consistent boundary between core version control and forge-specific features, even for well-intentioned contributions.

## In brief

The HTTP/SSL configuration documentation thread saw Andrew Carter acknowledge the need for more detailed platform-specific notes about `http.sslCertType` and `http.sslKeyType`, particularly regarding OpenSSL engine parameters and PKCS#11 token support, with plans for an updated patch. A bugfix addresses a long-standing file descriptor double-close issue in the bundle transport code, primarily affecting i686 Windows, by clarifying ownership semantics and preventing duplicate closes during normal operation.