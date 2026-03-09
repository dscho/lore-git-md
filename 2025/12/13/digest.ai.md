# Git Mailing List Digest - 2025/12/13

**The day in brief.** A moderately busy Saturday with 31 emails across 9 threads, dominated by the finalization of the submodule gitdir path encoding series and several documentation/build system improvements. The standout development is the v6 submission of the long-running submodule path configuration work, now rebased and refined with all major design questions resolved.

## Notable threads

**Submodule gitdir path encoding reaches consensus**  
Patrick Steinhardt's 10-patch series implementing runtime configuration for submodule gitdir paths has reached v6 with all architectural questions settled. The series introduces `extensions.submodulePathConfig` as a pure runtime option, rejecting build-time approach after extensive discussion rounds. The implementation now follows a clear three-phase approach: infrastructure (patches 1-6), encoding (7-9), and migration (10). Key features include a four-tier fallback system (plain path -> URL encoding -> digit appending -> hashed name), case-folding protection via RFC3986 encoding variants, and centralized path resolution through `submodule_name_to_gitdir()`. The series includes comprehensive test coverage and a migration command to ease adoption. With Junio Hamano's requirements for strict config validation met and all major feedback addressed, this appears ready for merging after its long gestation period.

**git replay documentation refined**  
Kristoffer Haugsbakk's v2 series improves the `git replay` man page with three focused documentation patches. The most substantive change clarifies the `--contained` option's behavior, replacing vague "advance all branches contained in" language with precise "update all branches that point at commits in" wording. The series also documents `git replay`'s silent conflict behavior and standardizes internal linking markup. These changes follow review feedback from Junio Hamano and Phillip Wood, with the v2 iteration showing careful attention to terminology and cross-referencing. While primarily documentation, the improvements help users understand this relatively new command's behavior.

**macOS iconv workaround refined**  
The ongoing macOS iconv handling thread saw two build system patches addressing the broken ISO-2022-JP conversion issue. The changes introduce a `NO_HOMEBREW` flag to control linking against Homebrew's libiconv while maintaining backward compatibility. The implementation now properly handles both Intel (`/usr/local`) and Apple Silicon (`/opt/homebrew`) Homebrew installations through new `HOMEBREW_*` variables. This provides a practical workaround for macOS's problematic system iconv while giving users/builders control over their environment. The changes follow the maintainer-endorsed direction from earlier discussion and appear ready for merging.

## In brief

**MEMZERO_ARRAY conversions continue** -- Toon Claes submits a 2-patch series replacing memset() calls with MEMZERO_ARRAY() in diffcore-delta.c, linear-assignment.c, and shallow.c, continuing the codebase standardization effort. The changes are mechanical but improve Coccinelle compliance.

**Test version string handling fixed** -- René Scharfe corrects how `t/t4014-format-patch.sh` handles Git version strings containing spaces, particularly addressing Apple Git distributions. The one-line change replaces space-based stripping with explicit prefix removal.

**Hook API modernization progresses** -- Adrian Ratiu reports his 11-patch hook API conversion series is ready for merging into 'next' after weeks without new feedback. The changes convert various Git hooks to the new API while adding output capture capabilities and child state helpers.

**Diff/merge behavior clarified** -- Elijah Newren explains an apparent inconsistency between `git diff` and `git merge` rename detection was actually due to comparing different commit ranges. When examining equivalent operations, both commands show the same 66% similarity threshold behavior.