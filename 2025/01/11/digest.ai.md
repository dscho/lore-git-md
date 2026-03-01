# Git Mailing List Digest — 2025/01/11

**The day in brief.** A moderately active Saturday with 19 emails across 13 threads, featuring continued debate on hash algorithm safety, security discussions around credential handling, and several practical improvements to authentication and configuration systems. The most notable developments include a proposed security hardening for URL credentials and progress on SMTP bearer authentication.

## Notable threads

### Hash algorithm safety debate continues

The long-running discussion about hash algorithm safety saw multiple technical exchanges today. Jeff King and Taylor Blau worked through edge cases in `hash_algo_by_ptr()` implementation, addressing NULL pointer handling and performance considerations. Meanwhile, Junio sought clarification about the preparatory patches' status, to which Taylor confirmed they've been squashed into the current series. The thread reveals competing approaches to type safety - Taylor's separate algorithm instances versus Jeff's nested function structs - with both sides now clearer on how their proposals intersect.

### Credential security discussions deepen

Multiple threads converged on credential security today. Brian m. carlson provided concrete counterexamples showing how Junio's proposed credential-flow detection would fail with Kerberos and TLS client certificate workflows. This led to consensus that any effective warning system would need to examine username strings directly, introducing complexity. Separately, Randall Becker noted this issue similarly affects multiple platforms (GitHub, BitBucket, GitLab) despite differing token structures, highlighting the challenge of finding a generic solution.

### SMTP bearer authentication advances

Julian Swagemakers' work on SMTP bearer authentication for `git send-email` saw positive testing results from Mirth Hickford, confirming both XOAUTH2 and OAUTHBEARER methods work with Gmail. The thread resolved an earlier concern about debug output formatting being merely a display artifact. A related patch from Julian implementing direct OAUTHBEARER/XOAUTH2 support (bypassing Perl's Authen::SASL limitations) also landed today, complete with configuration examples for major providers.

## In brief

Jeff King's combine-diff refactoring series saw patch 9 approved by Junio, simplifying code by inlining `path_appendnew()` into its single caller. The `help.autocorrect` configuration improvement reached v3, now properly handling boolean string values by mapping "false"/"off"/"no" to never showing guesses. A meson build fix addressed parallel documentation generation issues by adding missing dependencies on asciidoc configuration. Randall Becker questioned Git's behavior with missing config files, probing whether the current fail-early approach represents a security measure.

## On the radar

The proposed change to make `transfer.credentialsInUrl` default to "warn" rather than "allow" could spark discussion about balancing security and convenience. A regression report about `git describe` failing in mirrored repositories with multiple remotes may need investigation as a potential 2.48.0 regression. Seyi Chamber's Outreachy update on converting t-hash.c tests to the clar framework shows steady progress in test modernization.