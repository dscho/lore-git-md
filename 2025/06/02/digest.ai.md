# Git Mailing List Digest - 2025/06/02

**The day in brief.** A busy Monday with 160 emails across 33 threads, dominated by major refactoring work in the_repository removal and object database subsystem, along with significant feature additions to imap-send and git subtree. Key highlights include Patrick Steinhardt's v4 series completing the ODB refactoring, Aditya Garg's IMAP authentication improvements reaching v12, and a new GPG signing feature for git subtree.

## Notable threads

### Object Database Refactoring Completes

Patrick Steinhardt's v4 series (17 patches) finalizes the object database subsystem refactoring as part of the broader `the_repository` removal effort. The series systematically converts all ODB-related functions to use explicit `object_database` parameters with consistent `odb_` prefix naming, eliminating the last `the_repository` dependencies in this subsystem. Key changes include renaming core structures (`raw_object_store` → `object_database`, `object_directory` → `odb_source`), updating function signatures across 48+ files, and establishing proper parent pointers between ODBs and repositories. The mechanical but wide-ranging changes maintain all existing behavior while enabling future pluggable backends. Junio Hamano has begun reviewing the series which appears ready for integration after resolving trivial merge conflicts.

### IMAP Authentication Overhaul Finalized

Aditya Garg's v12 series brings OAuth2.0 support to `imap-send` while fixing several authentication edge cases. The implementation now handles both standard OAUTHBEARER and Google's XOAUTH2 methods across OpenSSL and libcurl backends, with thorough documentation for Gmail/Outlook configuration. The series also addresses a regression from Git 2.46.0 that broke basic functionality by incorrectly clearing folder configurations. After extensive review, the final version properly verifies local authentication method support before attempting connections and standardizes error message formatting. Junio's feedback on documentation phrasing and option handling has been fully incorporated, making this likely the last iteration before merging.

### GPG Signing Comes to git subtree

Patrik Weiskircher introduces GPG signing support for `git subtree` operations through a carefully structured 2-patch series. The implementation adds `-S/--gpg-sign` options to all commit-creating commands (add, merge, split, pull, push), passing the signing flag through to underlying `git commit-tree` and `git merge` calls. A preparatory patch modernizes the script's argument parsing using `--stuck-long` mode to cleanly handle optional key IDs. The feature includes 113 lines of new tests verifying signing behavior across various scenarios including explicit key IDs, `--squash` interactions, and all major subtree operations. This fills a notable gap in subtree's functionality while maintaining consistency with Git's core signing behavior.

### Batched Reference Update Fixes

Karthik Nayak addresses production issue in GitLab's deployment of batched reference updates with a 3-patch series. The fixes handle edge cases encountered at scale: a files backend segfault when skipping failed updates, test description cleanup, and directory/file conflicts in `git receive-pack`. The most substantive change modifies `receive-pack` to process deletions in a separate transaction first, mirroring the solution previously applied to `git fetch`. While this alters hook notification ordering (deletions now consistently precede creations), the thread concludes this tradeoff is acceptable given the improved reliability. The series has undergone thorough review with all technical concerns addressed.

### Maintenance Task Race Conditions

Patrick Steinhardt's v3 series (12 patches) systematically eliminates race conditions in Git's maintenance tasks by splitting operations into foreground (pre-detach) and background phases. The changes specifically address lockfile issues between reference and reflog operations during garbage collection, moving "pack-refs" and "reflog-expire" to the foreground while introducing a `--skip-foreground-tasks` flag for `git gc`. The series includes extensive refactoring to remove global state from task configuration and introduces a new `die(NULL)` pattern for cleaner error handling. With all feedback addressed and positive review from Karthik Nayak, this appears ready for integration.

## In brief

**Reftable test conversion** -- Justin Tobler completes the migration of reftable unit tests to the Clar framework across 10 patches, removing the custom test infrastructure while maintaining all coverage.

**BSD memory detection fix** -- Brad Smith corrects physical RAM reporting on OpenBSD/NetBSD systems by using `HW_PHYSMEM64` instead of the 4GB-limited `HW_PHYSMEM` sysctl.

**OpenBSD build updates** -- Brad Smith modernizes OpenBSD build configuration, removing obsolete workarounds for string functions available since OpenBSD 3.8/5.4 while adding required `DIR_HAS_BSD_GROUP_SEMANTICS`.

**Worktree ref verification** -- Shejialuo fixes `git refs verify` to handle older-format worktrees (pre-2.43.0) by ignoring missing refs` directories, with thorough test improvements from Kristoffer Haugsbakk and Eric Sunshine.

**cat-file batch mode extensions** -- Jacob Keller enhances `git cat-file --batch` with `%(objectmode)` formatting and improved submodule handling, building on test infrastructure updates for path-based object specs.

**IMAP folder listing** -- Final patch in Aditya Garg's series adds a `--list` option to `imap-send` showing available folders with RFC6154-compliant output, particularly useful for identifying Gmail/Outlook's non-standard folder names.

**Cygwin pathspec tests** -- Ramsay Jones marks 11 failing tests in t6137 as `!CYGWIN` due to the platform's unique handling of backslashes in quoted glob patterns, following investigation of the test failures.

**Signature handling docs** -- Christian Couder and Junio Hamano finalize documentation for `fast-export`'s experimental `--signed-commits` feature, clarifying its unstable status and potential future default changes.

## On the radar

**Rustification discussions** -- Ezekiel Newren's effort to introduce Rust code into Git remains active but awaits resolution of platform support concerns from Randall S. Becker regarding NonStop compatibility.

**Documentation restructuring** -- Jean-Noël Avila's ongoing conversion of man pages to synopsis-style AsciiDoc continues with steady progress, though not generating traffic today.

**Test modernization** -- Patrick Steinhardt's championing of the Clar unit test framework sees continued adoption, with the reftable conversion now complete and other subsystems likely to follow.