# Git Mailing List Digest — 2025/01/11

**The day in brief.** A moderately active Saturday with 19 emails across 13 threads, featuring continued security discussions around credential handling, progress on hash algorithm refactoring, and several practical improvements to authentication and configuration systems. The most notable developments include a proposal to change Git's default behavior for URL credentials and a regression report affecting bare repositories with multiple remotes.

## Notable threads

### Security hardening for URL credentials

A proposal from Jeff King suggests changing the default behavior of `transfer.credentialsInUrl` from "allow" to "warn", which would make Git emit warnings when credentials appear in plaintext in remote URLs. This security hardening measure builds on commit 6dcbdc0d66 which originally implemented the configuration option. The warning system would redact actual credentials while showing the problematic URL pattern, encouraging users toward safer credential storage methods like credential helpers. The thread includes discussion of escape hatches for cases where URL credentials remain necessary.

### Hash algorithm refactoring progresses

The ongoing hash algorithm refactoring work saw several technical exchanges between Taylor Blau and Jeff King regarding pointer safety in the `hash_algo_by_ptr()` function implementation. The discussion focused on edge cases in handling NULL pointer checks and the tradeoffs between different approaches to type safety. Junio C Hamano also received clarification about the version control status of the series, confirming that preparatory patches have been squashed into the current iteration. The thread shows steady progress toward eliminating separate unsafe hash variants while maintaining safety guarantees.

### SMTP bearer authentication implementation

Julian Swagemakers' work on SMTP bearer authentication for `git send-email` received positive testing feedback from Mirth Hickford, confirming successful operation with both XOAUTH2 and OAUTHBEARER methods on Gmail. The thread includes detailed testing instructions and resolves an earlier concern about debug output formatting. A separate but related patch from Julian implements direct handling of these authentication mechanisms since they aren't supported by Perl's Authen::SASL module, providing proper RFC-compliant implementations for major email providers.

### Regression in multi-remote bare repositories

A bug report highlights a regression in Git 2.48.0 affecting bare repositories with multiple remotes. The reporter demonstrates that after cloning with `--mirror` and adding a second remote, `git describe` fails with "fatal: Not a valid object name HEAD" when checking out a branch from the second remote. The issue appears related to commit 5f212684abb and affects users maintaining mirrored repositories with multiple remotes. The clear reproduction steps should help maintainers investigate whether this is indeed a regression from the referenced commit.

## In brief

Jeff King's combine-diff refactoring series saw patch 9 approved by Junio, simplifying code by inlining the `path_appendnew()` helper function while maintaining identical behavior. The OS version information discussion clarified that the current patch includes a `osVersion.command` configuration option as a middle ground between full configurability and system reliance, addressing Randall S. Becker's enterprise use case. A build system fix addresses parallel documentation generation in meson builds by adding proper dependencies on asciidoc configuration files. Randall Becker also raised a question about Git's behavior with missing configuration files, inquiring whether the current failure mode (rather than auto-creation) represents a deliberate security measure.

## On the radar

The security discussion around GitHub personal access tokens (PATs) remains unresolved, with Junio acknowledging that any effective warning system would need to examine username strings directly rather than relying on authentication flow patterns. This creates maintenance burdens as credential formats evolve across platforms, with no clear path forward that satisfies both security needs and compatibility requirements. The Outreachy intern's work on converting t-hash.c tests to the clar framework continues as part of the broader test modernization effort, with a recent blog post sharing conversion experiences.