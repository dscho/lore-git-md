# Git Mailing List Digest - 2025/12/13

**The day in brief.** A moderately busy Saturday with 31 emails across 9 threads, dominated by the finalization of the submodule gitdir path encoding series (v6) and several documentation/build system improvements. The submodule work represents a significant architectural change that's now ready for merging after extensive review.

## Notable threads

**Submodule gitdir path encoding reaches consensus** -- Patrick Steinhardt's 10-patch series implementing runtime configuration for submodule paths via `extensions.submodulePathConfig` has reached its sixth iteration with all major design questions resolved. The series establishes a four-tier fallback system (plain path -> URL encoding -> digit appending -> hashed name) to prevent filesystem collisions, with case-folding protection for case-insensitive systems. The implementation now follows a clear three-phase approach (infrastructure -> encoding -> migration) and includes a migration command to ease adoption. Ben Knoble raised the only remaining question - a clearer explanation of user benefits - which may prompt additional documentation but doesn't block the technical implementation.

**`replay` documentation improvements** -- Kristoffer Haugsbakk's v2 series (3 patches) refines the `git replay` man page based on prior feedback. The changes clarify the silent conflict behavior and better explain the `--contained` option's interaction with `--onto`. The series has addressed all substantive review comments and appears ready for merging, completing the documentation standardization effort started earlier this month.

**macOS iconv and Homebrew build fixes** -- Two patches address ongoing macOS build issues, adding a `NO_HOMEBREW` flag for build configuration control and implementing Homebrew-based workarounds for macOS's broken ISO-2022-JP conversion. The changes maintain backward compatibility while giving users more control over their build environment, following the maintainer-endorsed direction from earlier discussion.

**MEMZERO_ARRAY conversions continue** -- Junio Hamano and René Scharfe advanced the MEMZERO_ARRAY/Coccinelle effort with build system improvements and additional conversions in diffcore-delta.c, linear-assignment.c, and shallow.c. The thread revealed some remaining include path handling issues but demonstrated the rules working correctly in most contexts. The changes improve CI debuggability by showing failed Coccinelle patch contents rather than just indicating failure.

## In brief

**Test version string handling** -- René Scharfe fixes version string parsing in `t/t4014-format-patch.sh` to properly handle Git distributions (like Apple's) that include spaces in their version strings.

**Hook API modernization status** -- Adrian Ratiu reports his 11-patch hook API conversion series is ready for merging into 'next', having addressed all feedback and passed tests. This unblocks planned follow-up work on parallel/config hooks.

**Diff/merge rename detection explained** -- Elijah Newren clarified that an apparent inconsistency between `git diff` and `git merge` rename detection was actually due to comparing different commit ranges, not a bug. The commands behave consistently when comparing equivalent operations.