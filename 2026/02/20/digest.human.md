# Git Digest - 2026/02/20

## The day in brief

A busy day with 118 emails across 38 threads saw several significant developments. Key highlights include the finalization of Patrick Steinhardt's ref iteration API unification series, progress on config-based hooks from Adrian Ratiu, and multiple threads reaching maintainer approval. The day also featured technical discussions about platform-specific optimizations and ongoing work to remove `the_repository` global variable.

## Notable threads

### Ref iteration API unification complete

Patrick Steinhardt's 17-patch series to unify Git's ref iteration interfaces reached completion after extensive review iterations. The work replaces 14 specialized `refs_for_each_*()` functions with a single configurable `refs_for_each_ref_ext()` interface that uses an options structure to control behavior. The systematic approach - first introducing the new API, then migrating callers, and finally removing old interfaces - demonstrates Git's careful refactoring methodology. The changes touch core ref handling across multiple subsystems but maintain identical behavior while significantly reducing API surface area.

### Config-based hooks nearing integration

Adrian Ratiu's config-based hooks series progressed through detailed review, with Junio Hamano and Patrick Steinhardt providing feedback on implementation details. The v2 series (8 patches) introduces `hook.<name>.command`, `hook.<name>.event`, and `hook.<name>.enabled` configuration options while maintaining filesystem hook compatibility. Discussions focused on memory management patterns, disabled hooks visibility, and documentation clarity. The series is now in final polishing with all architectural decisions settled, awaiting only minor refinements before integration.

### Linux fsmonitor support ready

Junio Hamano signaled readiness to merge the Linux inotify-based fsmonitor implementation after confirming the remaining known memory leaks (512-byte at startup and 40-byte in IPC handling) aren't severe enough to block integration. The patch series has undergone four iterations with comprehensive testing, addressing Windows/MSVC compatibility and build system integration. This long-awaited feature brings Linux to parity with existing Windows and macOS filesystem monitoring support, enabling faster status operations through event watching rather than directory scanning.

### Submodule ignore behavior finalized

The submodule ignore consistency series addressing `git add` behavior with `ignore=all` configurations received maintainer approval after multiple iterations. The v4 implementation makes `git add` respect `submodule.<name>.ignore=all` by default (matching `git status` and `git diff`) while requiring explicit `--force` to override. The changes include comprehensive test coverage, documentation updates, and careful handling of edge cases like sparse checkout interactions. With all technical concerns resolved and Junio's approval, this thread appears ready for merging.

## In brief

**Reference backend selection** -- Karthik Nayak's series enabling zero-downtime migrations between files and reftable backends received final polish on test optimizations and is now maintainer-approved.

**MIDX compaction** -- Junio checked on Taylor Blau's 18-patch MIDX compaction series, which appears complete after addressing review feedback but has been quiet since January.

**Histogram diff fix** -- Junio pinged Yee Cheng Chin about the promised v2 patch fixing edge cases in the histogram diff algorithm's shifted group handling.

**Shallow fetch improvements** -- Samo Pogačnik's memory leak fix and relative-depth fetch corrections for shallow repositories are queued for integration after thorough review.

**MacOS credential helper** -- Koji Nakamaru's v4 patch standardizing the `git-credential-osxkeychain` build process completed the transition to top-level Makefile targets.

**Gitweb mobile** -- Junio signaled intent to merge Rito Rhymes' mobile responsiveness improvements given lack of objections to the v2 implementation.

**Submodule remote defaults** -- Final test portability fixes were applied to the series correcting submodule fetching behavior with non-"origin" remote names.

**ODB abstraction** -- Junio moved Patrick Steinhardt's ODB iteration unification to 'next' after confirming the implementation reached a stable "good enough" state.

**Mailmap refactoring** -- Burak Kaan Karaçay's GSoC contribution removing `the_repository` from mailmap handling was approved after addressing header inclusion issues.

**Format-patch docs** -- Kristoffer Haugsbakk's documentation improvements for `git-patch-id` batch processing are queued after D. Ben Knoble confirmed his feedback was addressed.

**On the radar**

**Replay revert interface** -- Junio expressed preference for using subcommands (`revert`, `pick`) rather than flags in `git replay`, potentially requiring interface restructuring of the already-implemented `--revert` feature.

**Send-email client certs** -- Initial patches adding SMTP client certificate support entered review, with Junio providing detailed documentation feedback on the new authentication options.

**Maintenance strategy change** -- Patrick Steinhardt's series switching the default maintenance strategy to geometric repacking completed test preparation and is now considering whether to gate the change behind `feature.experimental`.

**Branch prefix config** -- Junio noted new configuration options should typically be preceded by proven command-line flags, suggesting the proposed `branch.addCurrentBranchAsPrefix` needs more vetting before becoming a config setting.