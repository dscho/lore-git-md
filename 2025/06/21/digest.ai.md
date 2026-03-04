# Git Mailing List Digest - 2025/06/21

**The day in brief.** A moderately active Saturday with 13 emails across 8 threads, featuring a late-stage bug discovery in Taylor Blau's MIDX/cruft pack series, continued discussion on credential port handling, and Junio's monthly "What's cooking" status update. The most notable developments include a regression fix for submodule remote refactoring and substantive design discussions about Git's credential system architecture.

## Notable threads

**MIDX/cruft pack interaction bug** -- Jeff King reports a test failure in the final patch of Taylor Blau's performance optimization series when running with specific test environment variables (`GIT_TEST_MULTI_PACK_INDEX=1` and `GIT_TEST_MULTI_PACK_INDEX_WRITE_INCREMENTAL=1`). The issue causes cruft packs to incorrectly appear in MIDX files despite the new `repack.midxMustContainCruft` config. This late discovery in an otherwise well-reviewed series may require additional investigation before merging.

**Credential port handling debate** -- The discussion around git-credential-netrc's symbolic port support deepens, with Maxim Cournoyer proposing a robust Scheme implementation using `getservbyname` for port resolution. Brian Carlson counters with protocol documentation evidence that the credential system should only handle numeric ports, suggesting resolution should happen earlier in the call chain. The thread now presents three technical approaches: original regex validation, helper-side resolution, or caller-side conversion.

**Submodule remote refactoring regression** -- Lidong Yan identifies and fixes a segfault in the submodule remote lookup series caused by v3's removal of `branch->merge_name` while some code paths still relied on its null-check behavior. The concise fix updates `branch_has_merge_config()` to check `branch->set_merge` instead and uses `FREE_AND_NULL` in `merge_release()` to prevent double-free issues, resolving test failures while maintaining the series' goals.

**"What's cooking" June update** -- Junio Hamano's monthly status report highlights several significant developments: preparation for SHA-256 as default in Git 3.0 (`bc/use-sha256-by-default-in-3.0`), a new `git repo-info` subcommand (`lo/repo-info`), and revival of `git imap-send` with OAuth2.0 support (`ag/imap-send-resurrection`). The report also notes ongoing work like the `the_repository` removal effort and upcoming Git 2.51 release timeline with an August 18th target.

## In brief

**Batched reference updates follow-up** -- Karthik Nayak clarifies maintenance path for the now-merged v5 series, noting only the second patch (D/F conflict handling) would be eligible for future squashing into the base topic.

**Bulgarian git-gui translation** -- Johannes Sixt accepts Alexander Shopov's update (578 strings) while parallel discussions continue about po-file format standardization and string ordering stability.

**Coccinelle check enforcement** -- The thread reaches consensus on making `make coccicheck` fail when changes are suggested, with minor shell syntax debate (`=` vs `-eq`) settled by historical precedent.

**Code formatting workflows** -- Jeff King proposes a rebase-based approach using `git clang-format` as an alternative to Christian Couder's format-patch integration idea, emphasizing local commit modification over patch-time checking.