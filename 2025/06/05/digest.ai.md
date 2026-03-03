# Git Mailing List Digest - 2025/06/05

## The day in brief

A busy day with 111 emails across 24 threads, dominated by major refactoring work in the object database subsystem and critical bugfixes for the upcoming 2.50.0 release. Key developments include Patrick Steinhardt's v5 series removing `the_repository` from the ODB code, resolution of a critical NO_TCLTK build regression, and final polish on the `imap-send` OAuth2.0 support. Junio's "What's cooking" email provides a comprehensive snapshot of the project's current state.

## Notable threads

### Object database refactoring reaches v5

Patrick Steinhardt's 17-patch series to refactor Git's object database subsystem reached its fifth iteration, systematically converting the code to use explicit `object_database` parameters instead of the global `the_repository`. The changeset renames core structures (`raw_object_store` → `object_database`), standardizes function names with an `odb_` prefix, and reorganizes files to match the `refs.h`/`refs/` pattern. The work touches 140 files but remains mechanical in nature, building on prior review feedback to establish clean foundations for future pluggable backends.

### Critical NO_TCLTK build regression identified

A serious regression in Git 2.50.0-rc1's build system was traced to faulty TCL file handling when building without TCL/Tk support (`NO_TCLTK`). Johannes Sixt pinpointed the issue in `generate-tclindex.sh` where `$@` was incorrectly interpreted as shell syntax rather than Makefile syntax, causing source files to be deleted during failed builds. The thread progressed from initial tarball corruption suspicions to identifying the root cause, with a temporary workaround (`make NO_TCLTK=NoThanks`) suggested until the proper fix lands.

### imap-send OAuth2.0 support nears completion

The comprehensive `imap-send` overhaul addressing configuration regressions and adding OAuth2.0 authentication reached v13, with a critical logic error in the libcurl setup fixed. Jeff King identified that password handling for OAuth methods used incorrect `||` operators instead of `&&`, potentially leaking credentials. The series now supports both RFC-standard OAUTHBEARER and Google's XOAUTH2 protocols across OpenSSL and libcurl backends, along with new features like folder listing and improved credential prompts.

### Batched reference updates get edge case fixes

Patrick Steinhardt's v2 series addressed two edge cases in batched reference updates: a segfault in the files backend when skipping failed updates, and filesystem directory/file conflicts in `receive-pack`. The solution processes deletions in a separate transaction phase (`PHASE_DELETIONS`) before other updates, maintaining the performance benefits of batching while avoiding conflicts. Karthik Nayak's review suggested optimizations for empty transactions, with the changes now ready for integration.

## In brief

**Reftable test conversion complete** -- Seyi Kuforiji's 10/10 patch series finished converting all reftable tests to the Clar framework, removing the transitional `lib-reftable-clar` helpers in the final cleanup.

**Memory leak in pack-bitmap boundary traversal** -- A follow-up patch fixed remaining conditional-free issues in `find_boundary_objects()` that could leak bitmaps during successful operations.

**curl typecheck warnings fully addressed** -- Johannes Schindelin's supplemental patch fixed additional boolean flag cases (`CURLOPT_UPLOAD` etc.) that triggered warnings on macOS, completing Jeff King's earlier series.

**Windows/Meson test hang workaround** -- Johannes Schindelin added a `TEE_DOES_NOT_HANG` prerequisite to skip affected tests on buggy MSYS2 runtime versions until CI updates to Git for Windows 2.50.0.

**Stash import/export final review** -- Phillip Wood provided last feedback on the mature stash import/export series, noting an inconsistency between `--print` and `--to-ref` option handling that needs resolution.

**git show --merge memory leak** -- Junio proposed a new test case for conflicted index states after a leak was fixed in `prepare_show_merge()`, improving coverage for this code path.

## On the radar

**JavaScript diff driver proposal** -- Derick W. de M. Frias's initial submission for JavaScript/TypeScript diff support received style feedback, marking the latest attempt at this recurring but uncompleted feature.

**git config get version confusion** -- User reports highlighted documentation ambiguity around the `get` subcommand syntax introduced in Git 2.46.0, with older versions producing confusing error messages.

**Maintenance task refactoring** -- Patrick Steinhardt's series to improve maintenance command reliability awaits final documentation updates about `--task`/`--schedule` mutual exclusion before merging.