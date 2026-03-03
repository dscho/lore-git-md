Here's the daily digest for May 28, 2025:

**The day in brief.** A busy day with 93 emails across 24 threads, featuring significant work on MIDX optimizations, IMAP authentication improvements, and the release of Git v2.50.0-rc0. Key highlights include Taylor Blau's finalized MIDX/cruft pack series and Aditya Garg's comprehensive IMAP authentication overhaul reaching v7.

**Notable threads**

**MIDX optimization series finalized**  
Taylor Blau's 9-part performance optimization series for MIDX and cruft pack interaction has reached its final form (v4). The series introduces configurable control over whether cruft packs must be included in MIDX files during repacking, with a new `repack.midxMustContainCruft` option. The changes have been validated in GitHub's production environment and address all prior review comments, including a scope refinement spotted by Elijah Newren. The series now appears ready for merging after several iterations of careful review.

**IMAP authentication overhaul complete**  
Aditya Garg's comprehensive IMAP series (now at v7) addresses multiple authentication issues while adding modern OAuth2.0 support. The changes fix critical configuration parsing bugs, implement RFC-compliant PLAIN authentication, add comprehensive OAuth2.0 support (including Google's XOAUTH2 variant), and fix all identified CRAM-MD5 memory leaks. New user-facing features include a `--folder` CLI option and persistent backend selection via `imap.usecurl`. The series has undergone thorough review with input from Jeff King and Junio Hamano.

**Git v2.50.0-rc0 released**  
Junio Hamano announced the first release candidate for Git 2.50, containing 541 non-merge commits from 73 contributors. Notable changes include new cruft pack controls, TCP keepalive for HTTP, machine-parsable rev-list output, and continued work to reduce `the_repository` usage. The release also includes numerous performance improvements and the deprecation of the merge-recursive backend.

**Signature handling marked experimental**  
Junio Hamano marked `fast-export`'s `--signed-commits` feature as experimental and changed its default behavior from 'abort' to 'strip' for signed commits. This follows discussion about signature verification workflows and maintains backward compatibility while the feature matures. Elijah Newren raised questions about the default change process, with Junio clarifying this is a special case affecting only pre-release code.

**In brief**  

**Documentation standardization** -- Jean-Noël Avila confirmed his `git-verify-*` documentation updates were merged in v2.50.0-rc0, completing another piece of the ongoing synopsis format conversion effort.

**Subtree GPG signing** -- A new contributor submitted a 2-part series adding `-S/--gpg-sign` support to `git subtree`, with comprehensive test coverage verifying signing works for all commit-creating operations.

**Test infrastructure fixes** -- Patrick Steinhardt addressed TAP output handling issues in the test suite, particularly around shell trace output that could corrupt TAP parsers when using verbose flags.

**Maintenance ref lock races** -- Patrick Steinhardt confirmed sending his series to fix race conditions between maintenance and ref-modifying operations, following up on earlier discussion with Emily Shaffer.

**NonStop build failure** -- Randall Becker reported a compilation error in Git 2.50.0-rc0's reftable code on NonStop systems, related to `REFTABLE_UNUSED` macro handling.

**On the radar**  

**Commit -A proposal** -- Jon Forrest's proposal to add a `-A` option to `git commit` (matching `git add -A` behavior) continues discussion, with Junio Hamano emphasizing the importance of maintaining backward compatibility for existing `-a` workflows.

**Index optimization exploration** -- Junio Hamano and Jon Forrest discussed potential hash table optimizations for Git's index, identifying tree generation requirements as a key constraint for any implementation.