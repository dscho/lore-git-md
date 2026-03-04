# Git Mailing List Digest - 2025/06/21

**The day in brief.** A moderately active Saturday with 13 emails across 8 threads, featuring Junio's monthly "What's cooking" report, a late-stage bug discovery in Taylor Blau's MIDX/cruft pack series, and ongoing discussions about credential port handling and code formatting workflows. The most notable development is Jeff King's identification of a test failure in what was thought to be a ready-to-merge performance optimization.

## Notable threads

**MIDX/cruft pack test failure surfaces** -- Jeff King reports a test failure in the final patch of Taylor Blau's series optimizing MIDX and cruft pack interaction. The issue occurs when running tests with both `GIT_TEST_MULTI_PACK_INDEX=1` and `GIT_TEST_MULTI_PACK_INDEX_WRITE_INCREMENTAL=1` set, where cruft packs incorrectly appear in the MIDX despite configuration to exclude them. This late discovery in an otherwise well-reviewed series (previously approved by both Junio and Elijah Newren) may require additional investigation before merging.

**Credential port handling debate continues** -- The discussion around symbolic port names in git-credential-netrc deepens, with Maxim Cournoyer demonstrating a Scheme implementation using `getservbyname` for proper port resolution. Brian Carlson counters by pointing to git-credential protocol documentation that refers specifically to "port numbers," arguing port resolution should happen before the credential protocol is involved. The thread now presents three technical approaches: original regex validation, helper-side resolution, or caller-side conversion.

**"What's cooking" for June 2025** -- Junio Hamano's monthly status update highlights several significant developments: preparation for SHA-256 as the default hash algorithm in Git 3.0 (`bc/use-sha256-by-default-in-3.0`), a new `git repo-info` subcommand (`lo/repo-info`), and revival of `git imap-send` with OAuth2.0 support (`ag/imap-send-resurrection`). Ongoing work includes the major `the_repository` removal effort (`ps/object-store`) and performance improvements for ref updates (`kn/fetch-push-bulk-ref-update`). The email also notes the upcoming 2.51 release schedule with RC periods in early August.

## In brief

**Submodule remote refactoring fix** -- Lidong Yan addresses a segfault in the submodule remote lookup series by fixing null pointer handling in `branch_has_merge_config()`, ensuring compatibility with the removal of `branch->merge_name` in v3.

**Bulgarian git-gui translation update** -- Johannes Sixt accepts Alexander Shopov's Bulgarian translation update for git-gui (578 strings) while parallel discussions continue about improving .po file format and string ordering.

**Batched reference updates follow-up** -- Karthik Nayak clarifies maintenance path for the now-merged series, noting only the second patch (addressing D/F conflicts in `git receive-pack`) would be eligible for future squashing into the base topic.

**Coccinelle check behavior change** -- Consensus solidifies around making `make coccicheck` fail when changes are suggested, with minor debate about shell syntax (`=` vs `-eq`) for the test condition now settled in favor of `=` based on historical precedent.

## On the radar

**Code formatting workflows** -- Jeff King proposes a rebase-based approach using `git clang-format` as an alternative to Christian Couder's format-patch integration idea, highlighting the challenge of balancing automation benefits with imperfect tool output.