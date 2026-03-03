# Git Digest: 2025/06/05

**The day in brief.** A busy day with 111 emails across 24 threads, dominated by major refactoring work in the object database subsystem and critical bugfixes for the upcoming 2.50.0 release. Key highlights include Patrick Steinhardt's v5 series removing `the_repository` from the ODB code, build system fixes for the NO_TCLTK regression, and final polish on the stash import/export feature. The Git project also saw its first "What's cooking" report since the 2.50.0-rc1 release, providing a comprehensive snapshot of the development pipeline.

## Notable threads

### Object database refactoring reaches v5

Patrick Steinhardt's 17-patch series to remove `the_repository` global state from Git's object database (ODB) subsystem reached its fifth iteration today. This foundational work systematically converts the ODB code to use explicit `object_database` parameters rather than relying on globals, while establishing consistent `odb_`-prefixed API naming. The series renames core structures (`raw_object_store` → `object_database`), moves implementation files (`object-store.{c,h}` → `odb.{c,h}`), and updates over 140 files with mechanical but thorough conversions. Reviewers including Derrick Stolee have signed off on the approach, with v5 addressing naming refinements and merge conflicts with other ongoing work. This represents a major step toward enabling pluggable ODB backends while improving code organization.

### Build system regression in NO_TCLTK

A critical build system regression affecting NO_TCLTK configurations dominated discussion today, with Randall Becker and Johannes Sixt identifying the root cause in `generate-tclindex.sh`. The bug, introduced in Git 2.50.0-rc1, causes the script to incorrectly delete TCL source files when tclsh is unavailable, rather than just cleaning up tclIndex. The issue stems from incorrect interpretation of `$@` as a shell variable rather than Makefile target reference. Junio Hamano acknowledged the oversight and suggested temporary workarounds (`make NO_TCLTK=NoThanks`) while the proper fix is finalized. This regression affects all platforms building without TCL/Tk support and represents a high-priority fix needed before the 2.50.0 release.

### Stash import/export nears completion

Phillip Wood provided final review feedback on the long-running stash import/export series, now at v7. The discussion focused on two remaining polish items: enforcing documented mutual exclusion between `--print` and `--to-ref` options, and clarifying ownership semantics in `write_commit_with_parents()`. With cross-platform validation from Ramsay Jones and all major technical concerns addressed, this feature appears ready for merging once these last documentation and code style items are resolved. The implementation introduces new `stash export`/`import` subcommands that store commit chains under `refs/stash-export/` while preserving message fidelity and stash stack behavior.

### Batched reference updates get edge case fixes

Karthik Nayak and Patrick Steinhardt collaborated on v2 fixes for edge cases in batched reference updates. The series addresses a files backend segfault when skipping failed updates and restructures `receive-pack`'s handling of file/directory conflicts using explicit `PHASE_DELETIONS`/`PHASE_OTHERS` processing. Review feedback led to replacing an `only_deletions` flag with clearer phase enumeration, improving readability concerns raised in v1. Test coverage in `t1400-update-ref.sh` and `t5516-fetch-push.sh` verifies the fixes work for real-world scenarios encountered at GitLab while maintaining batched update performance benefits.

### IMAP OAuth2 authentication refined

The IMAP OAuth2 series saw extensive discussion about authentication method handling, prompted by Jeff King's discovery of a critical logic error in the libcurl setup. The original implementation incorrectly used `||` between `strcmp()` checks for OAuth methods, making the condition always true. v13 corrected this to properly skip password setting for OAuth flows using `&&`. Junio Hamano then raised design questions about long-term maintainability, suggesting either BUG/die() for unsupported methods or explicit warnings about fallback to plain authentication. Aditya Garg explained the technical rationale behind the current implementation while showing willingness to adjust based on maintainer preferences. The series now awaits final resolution of these design questions before merging.

## In brief

**Reftable test modernization** -- Seyi Kuforiji's 10-patch series converting all reftable tests to the Clar framework reached v4, with final cleanup removing transitional helper files. Junio noted minor whitespace issues needing correction before merge.

**Memory leak fixes** -- Lidong Yan's v2 patches address leaks in `repo_logmsg_reencode()` callers, with Jeff King confirming the fixes work using a specialized test hack (`GIT_TEST_ALWAYS_REENCODE`).

**cURL typecheck warnings** -- Johannes Schindelin submitted additional fixes for boolean flag parameters (`CURLOPT_UPLOAD` etc.) that triggered warnings in macOS CI, complementing Jeff King's earlier series.

**Windows CI test hangs** -- Johannes Schindardt proposed skipping two `receive-pack` tests in Windows/Meson CI via `TEE_DOES_NOT_HANG` prerequisite, working around an MSYS2 runtime bug until GitHub runners update to Git for Windows 2.50.0.

**`git last-modified` documentation** -- Toon Claes addressed Justin Tobler's review feedback by clarifying default behavior and fixing include ordering in the experimental command.

**`git config get` version confusion** -- Kristoffer Haugsbakk clarified the `get` subcommand syntax was introduced in Git 2.46.0, explaining a user's confusion with older versions that require `--get` flag.

## On the radar

**Maintenance task refactoring** -- Patrick Steinhardt's series to improve maintenance command reliability awaits Derrick Stolee's suggested documentation updates about `--task`/`--schedule` mutual exclusion.

**JavaScript diff driver** -- Derick W. de M. Frias's initial submission for JavaScript/TypeScript diff support received preliminary style feedback from Johannes Sixt, who pointed to four prior attempts for reference.

**`commit.signoff` rejection** -- The thread concluded with Collin Funk endorsing Junio's decision by drawing parallels to FSF copyright assignment processes, emphasizing the legal significance of manual signoffs.