Here's the daily digest for May 28, 2025:

**The day in brief.** A busy day with 93 emails across 24 threads, featuring significant work on performance optimizations for MIDX and cruft packs, documentation improvements, and the release of Git v2.50.0-rc0. Key highlights include Taylor Blau's finalized MIDX optimization series and a comprehensive IMAP authentication overhaul reaching its seventh iteration.

**Notable threads**

**MIDX and cruft pack optimizations finalized**  
Taylor Blau's 9-part performance optimization series for MIDX and cruft pack interaction during repacking has reached its final polishing stage. The series introduces configurable control over whether cruft packs must be included in MIDX files, with a new `repack.midxMustContainCruft` option (default true). The changes have been validated in GitHub's production environment and address several edge cases around reachability and delta compression. The series has received positive reviews from Junio Hamano and Elijah Newren, with only minor documentation tweaks remaining before merging.

**IMAP authentication overhaul (v7)**  
A comprehensive series fixing `imap-send` authentication reaches its seventh iteration, now with final polish to user-facing messages. The changes include OAuth2.0 support (both RFC-standard OAUTHBEARER and Google's XOAUTH2), PLAIN authentication for OpenSSL, CRAM-MD5 memory leak fixes, and new configuration options like `--folder` and `imap.usecurl`. The series has undergone extensive review and appears ready for merging, representing a significant modernization of Git's email submission capabilities.

**Git v2.50.0-rc0 released**  
Junio Hamano announced the first release candidate for Git 2.50, containing 541 non-merge commits from 73 contributors. Notable changes include new repack options, TCP keepalive for HTTP, machine-parsable rev-list output, continued `the_repository` removal work, and numerous performance improvements. The release also marks the deprecation of the merge-recursive backend and includes significant reftable API updates.

**Signature handling marked experimental**  
Junio Hamano marked the `--signed-commits` feature in `fast-export`/`fast-import` as experimental and changed the default behavior from 'abort' to 'strip' for signed commits. This follows discussion about signature verification workflows and maintains backward compatibility while the feature continues to evolve. The change affects only pre-release code, avoiding the need for a full deprecation cycle.

**In brief**  

**MIDX API improvements** -- Taylor Blau refactors the MIDX pack lookup API to return `packed_git` pointers directly rather than pack IDs, making the interface more ergonomic and safer for MIDX chain handling.

**Test directory normalization** -- Mark Mentovai fixes test failures on macOS by normalizing working directory paths in test infrastructure, following Junio Hamano's suggestion to handle this at the system level rather than in individual tests.

**Subtree GPG signing** -- A new contributor adds `-S/--gpg-sign` support to `git subtree` commands, with comprehensive test coverage verifying signed commits across all operations.

**send-email threading fix** -- Aditya Garg corrects a counter issue in `git send-email` that could break message threading when editing the first message in a thread with `--in-reply-to`.

**Promisor remote optimization** -- A patch removes an expensive object verification check in `promisor_remote_get_direct()` that was causing slowdowns in large repositories.

**On the radar**  

**Maintenance ref lock races** -- Patrick Steinhardt's series to fix race conditions between maintenance and ref-modifying operations is awaiting final review after typo fixes were addressed.

**Commit -A proposal** -- Discussion continues about adding a `-A` option to `git commit` that would stage both modified and new files, with Junio Hamano emphasizing the need to preserve backward compatibility for existing `-a` behavior.