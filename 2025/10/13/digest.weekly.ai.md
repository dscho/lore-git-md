# Git Mailing List Weekly Digest - 2025/10/13 -- 2025/10/19

**The week in brief.** A busy week with 565 emails across 144 threads saw significant progress on multiple fronts. The standout developments include the finalization of atomic reference updates for `git replay`, major advancements in SHA-1/SHA-256 interoperability work, and security hardening against malicious hooks. Taylor Blau's massive repack refactoring series concluded review, while documentation improvements and test modernization continued steadily. The week also featured active discussions around AI contribution policies, `git whatchanged` deprecation, and Rust integration.

## Key developments

### Atomic reference updates for `git replay` finalized

Siddharth Asthana's series to make `git replay` perform atomic reference updates by default reached completion after multiple review iterations. The implementation now uses Git's ref transaction API to ensure all-or-nothing behavior while maintaining backward compatibility through a new `--ref-action` option and `replay.refAction` config. The comprehensive test suite verifies atomic behavior through lock file checks and includes thorough coverage of config parsing and CLI override scenarios. This enhancement significantly improves `git replay`'s reliability for complex history rewriting operations.

### SHA-1/SHA-256 interoperability moves forward

Brian M. Carlson's 9-patch SHA-1/SHA-256 interoperability series was approved and moved to 'next', marking a major milestone in the multi-year effort to enable hash algorithm compatibility. The series includes foundational documentation updates (pack format specifications, loose object format specs), a new `rev-parse --show-object-format=compat` plumbing command, and enhanced `fsck` validation for GPG signature headers. With only minor documentation tweaks since v2 and no outstanding technical concerns, this work lays crucial groundwork for Git 3.0's planned SHA-256 transition.

### Security series against malicious hooks

Michael Lohmann submitted a 5-part series addressing arbitrary code execution via Git hooks in untrusted repositories. Building on prior `safe.directory` work, the patches introduce both temporary (`--allow-unsafe` flag) and persistent (`safe.assumeUnsafe` config) mechanisms to prevent automatic trust of repositories owned by the current user. The implementation includes comprehensive tests and thorough documentation of the security implications, following Git's standard practice of separating refactoring from behavior changes. This represents an important hardening of Git's security model for shared environments.

### Repack modularization completed

Taylor Blau's 49-part refactoring series to modularize `builtin/repack.c` concluded review and received maintainer approval. The changes systematically eliminate global variables like `the_repository` and `the_hash_algo` while splitting the monolithic repack implementation into dedicated compilation units for cruft packs, filtered packs, MIDX handling, and geometry calculations. Reviewers praised the careful balance between interface improvements and practical constraints, with the changes laying groundwork for future incremental MIDX functionality while significantly improving the codebase's maintainability.

### Rust CI infrastructure approved

Patrick Steinhardt's v3 series establishing comprehensive Rust CI infrastructure received final approval. The 6-patch set adds rustfmt formatting checks, Clippy static analysis, MSRV validation, and completes Windows support through userenv.dll linking. After addressing all review feedback across three versions, the series is now ready to merge, marking a significant milestone in Git's Rust integration efforts. The implementation carefully balances Rust ecosystem standards with Git's existing CI patterns, including an accepted deviation from Git's 80-column limit for Rust code.

## In brief

**Reftable optimization infrastructure** -- Karthik Nayak and Patrick Steinhardt finalized a 9-part series introducing a `--check-for-auto` flag to check if reference optimization is needed without performing it.

**Fast-import/fast-export signature handling** -- Christian Couder completed implementation of symmetric `--signed-tags=<mode>` support across both commands.

**Documentation formatting fixes** -- Jeff King's fixes for nested list rendering issues in reftable format documentation were merged into 'next'.

**Test modernization** -- Multiple patches updated tests to use modern helpers (`test_path_is_file`, `test_path_is_dir`) instead of direct shell checks.

**Patch-id hash algorithm enforcement** -- Clarified that `git patch-id` must always use SHA1, removing outdated code while explicitly setting the hash algorithm.

**GPG/SSH line ending fix** -- Okhuomon Ajayi addressed CR/LF handling in signature verification, renaming `remove_cr_after()` to `trim_cr_before_lf()`.

**gitk window geometry persistence** -- Johannes Sixt finalized a two-part series restoring full window geometry saving in gitk.

**Git for Windows security release** -- Version 2.51.1 was announced, addressing high-severity CVE-2025-26625 in Git LFS.

**git-reset documentation overhaul** -- Julia Evans completed a 4-part series reorganizing the man page to better match user workflows.

**Bash completion improvements** -- KIYOTA Fumiya added `--exclude=` completion for `git log`/`shortlog`.

## Looking ahead

**Git 3.0 planning** -- Brian m. carlson's timeline discussion for SHA-256 transition continues, with current focus on gathering ecosystem readiness reports.

**Rustification effort** -- Ezekiel Newren's Rust integration work remains active but faces platform support concerns from Randall S. Becker regarding NonStop compatibility.

**`the_repository` removal** -- Outreachy applicants are being onboarded to this ongoing effort, with guidance to examine `builtin/` files for refactoring opportunities.

**Geometric repack strategy** -- Patrick Steinhardt and Taylor Blau discussed future enhancements to the new maintenance strategy, including configuration options for the geometric split factor.