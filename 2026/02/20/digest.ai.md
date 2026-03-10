# Git Mailing List Digest - 2026/02/20

**The day in brief.** A busy Friday with 118 emails across 38 threads saw significant progress on several fronts: reference storage backend selection reached final approval, `git replay` gained revert capability, and the `the_repository` removal effort advanced in multiple subsystems. Notable discussions included interface design for configurable hooks and a major change to `git maintenance`'s default strategy.

## Notable threads

### Reference storage backend selection finalized

Karthik Nayak's series implementing configurable reference storage backends (files<->reftable) received maintainer approval after seven iterations. The v7 patches complete the implementation allowing zero-downtime migrations between backends through three configuration mechanisms: `extensions.refStorage` config, `GIT_REFERENCE_BACKEND` environment variable, and URI syntax. Final polish addressed test optimizations and documentation clarifications, with Junio Hamano indicating readiness to merge to 'next'. This represents a major milestone for GitLab's large repository migration needs.

### `git replay` gains revert capability

Adrian Ratiu's config-based hooks series progressed through detailed review, with v2 addressing feedback about debuggability and memory management patterns. The implementation now tracks config scope (global vs repository) and consolidates cleanup functions while maintaining filesystem hook compatibility. Junio Hamano and Patrick Steinhardt provided extensive review, focusing on making the interface more maintainable and user-friendly. The series is in final polishing with all 8 patches functionally complete.

### Maintenance strategy change

Patrick Steinhardt proposed changing `git maintenance`'s default strategy from "gc" to "geometric" in an 8-patch series. The first 7 patches comprehensively prepare the test suite for the behavioral change by adding new test variables and adjusting maintenance configurations. The final patch makes the one-line strategy switch, citing geometric repacking's advantages for large repositories while maintaining backward compatibility through config overrides. This carefully staged approach demonstrates Patrick's characteristic thoroughness in managing impactful changes.

## In brief

**Submodule ignore behavior** -- Claus Schneider's series aligning `git add` with `status`/`diff` for `ignore=all` submodules is technically complete after addressing all review feedback, now awaiting final confirmation before integration.

**MIDX compaction** -- Junio followed up on Taylor Blau's 18-patch series implementing `git multi-pack-index compact`, noting it hasn't seen activity since addressing earlier review feedback and may be ready for merging.

**Histogram diff fix** -- Junio pinged Yee Cheng Chin about the promised v2 patch fixing edge cases in the histogram diff algorithm's handling of matching groups, with technical approach already established.

**Shallow fetch improvements** -- Samo Pogačnik's fixes for memory leaks and relative-depth fetching in shallow repositories received maintainer approval after five iterations with comprehensive test coverage.

**Linux fsmonitor** -- Junio checked on the status of the inotify-based Linux filesystem monitoring implementation, which addressed all issues except two minor memory leaks that may not block merging.

**Submodule fetch behavior** -- Junio noted a test portability issue in a series fixing submodule fetching with non-"origin" remotes, otherwise ready with all substantive concerns addressed.

**ODB abstraction** -- Junio marked Patrick Steinhardt's ODB abstraction series (v4 14/14) as ready for 'next', completing the introduction of `odb_for_each_object()` as a unified interface.

**Symlinked .git handling** -- Tian Yuchen's bugfix allowing symlinked `.git` directories when pointing to valid repositories reached v8 with comprehensive security considerations and test coverage.

**Format-patch documentation** -- Kristoffer Haugsbakk's documentation improvements for `git-patch-id` batch processing completed review and await merging, adding practical examples and performance data.

**Gitweb mobile responsiveness** -- Rito Rhymes' mobile-friendly CSS changes for gitweb received maintainer approval after addressing Eric Sunshine's organization feedback in v2.

**macOS credential helper** -- Koji Nakamaru finalized build system improvements for `git-credential-osxkeychain` in v4, properly integrating dependency directory handling with the main Makefile.

**Send-email client certs** -- David Timber proposed adding certificate authentication support to `git send-email`, with initial patches adding config options for client certificates and private keys.

**Branch prefix config** -- Yoann Valeri introduced a `branch.addCurrentBranchAsPrefix` option to automatically prefix new branch names, though Junio suggested first validating the concept via command-line flags.

**Ref iteration API unification** -- Patrick Steinhardt began a major refactoring series (17 patches) to replace Git's 14 specialized ref iteration functions with a single configurable `refs_for_each_ref_ext()` interface.

## On the radar

**Rustification effort** -- Ezekiel Newren's work to introduce Rust code into Git continues, though platform support concerns from Randall S. Becker remain unresolved.

**Hook parallel execution** -- brian m. carlson raised important output handling considerations for Adrian Ratiu's future work on parallel hook execution, particularly around TTY preservation.

**Cover letter formatting** -- Jeff King and Junio discussed improving `git format-patch`'s cover letter commit listing format, suggesting `log --oneline --reverse` as a more readable alternative.