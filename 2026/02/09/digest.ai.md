Here's the Git mailing list digest for February 9, 2026:

## The day in brief

February 9, 2026 was a high-volume day with 137 emails across 35 threads, dominated by technical discussions around several major ongoing efforts: the `the_repository` removal project, reference backend selection, Rust infrastructure, and security hardening for `git am`. Key developments include final polish on Karthik Nayak's ref backend selection series, progress on UTF-8 alias support, and Junio Hamano's approval of Kristoffer Haugsbakk's `git am` documentation improvements.

## Notable threads

**Ref backend selection reaches final polish** -- Karthik Nayak's series implementing configurable backend selection (files<->reftable) is in its final stages after extensive review. The v5 iteration now handles both persistent (`extensions.refStorage` config) and ephemeral (`GIT_REFERENCE_BACKEND` env var) specifications while properly supporting worktree references. Patrick Steinhardt's thorough review identified only minor documentation nits around terminology ("payload" vs "location") and validation edge cases. Junio has signaled approval of the core design, with the series awaiting these final tweaks before integration.

**UTF-8 alias design converges** -- After initial concerns about Unicode normalization complexities raised by Theodore Tso, the thread converged on Jeff King's proposal for a three-level config syntax (`alias.foo.command`) that avoids direct Unicode handling. Jonatan Holmgren's implementation uses config subsections to preserve case and UTF-8 characters while maintaining backward compatibility. Junio endorsed the approach, noting it leverages macOS's existing `precompose_argv_prefix()` for normalization where needed. The patch includes comprehensive tests for international characters and documents the new syntax alongside the old.

**`git am` security documentation queued** -- Kristoffer Haugsbakk's documentation warnings about patch roundtrip hazards have been queued by Junio after incorporating feedback from Phillip Wood. The new `format-patch-caveats.adoc` explains how certain commit message patterns (unindented diffs, "---" separators, "Index:" lines) can be misinterpreted during patch application. This complements Phillip Wood's security-focused commit-msg hook changes and Patrick Steinhardt's proposed `--accept-ambiguous-patch` flag, forming a layered defense against accidental patch misapplication.

**Meson build system progress** -- Multiple threads advanced Git's Meson build system integration. Patrick Steinhardt confirmed the gitk/git-gui wiring is complete pending an upstream gitk PR, while D. Ben Knoble and Patrick collaborated on fixing `config-list.h` dependency tracking. Ramsay Jones reported a deprecation warning on Linux that may need follow-up. These changes represent incremental progress in the multi-year effort to modernize Git's build infrastructure while maintaining compatibility with existing Make-based workflows.

**Worktree API tensions in `the_repository` removal** -- Shreyansh Paliwal's effort to remove `the_repository` from `wt-status.c` surfaced deeper architectural questions about worktree handling. The discussion with Karthik Nayak and Junio Hamano revealed inconsistencies in how main worktrees are identified (NULL `id` in struct vs `\` marker in ref stores). This has become a blocking issue for the mechanical conversions, requiring potential API redesign before the status code can fully eliminate its repository globals. The thread shows how incremental refactoring can expose foundational design questions needing resolution.

**Rust directory structure proposed** -- Mike Hommey formally proposed moving Rust files to a dedicated `rust/` subdirectory after earlier RFC discussion. The patch follows revelations that mrustc can handle non-standard layouts, weakening one objection to reorganization. The change is mechanical (updating Makefile and meson.build references), but represents an important step in establishing Rust's place in the codebase. The separate git-cinnabar build issues that originally motivated the change remain unresolved but are now considered distinct from the structure question.

## In brief

**TRACE2 process ancestry for macOS** -- Matthew John Cheetham's series adding macOS support to Git's process tracing is nearly ready, with Derrick Stolee's review confirming the technical approach. The implementation uses `sysctl` with `KERN_PROC` to gather parent process info, matching Linux's format.

**Submodule ignore behavior finalized** -- Claus Schneider's v4 series making `git add` handle `ignore=all` submodules consistently with other commands is complete, with only a minor sign-off line formatting note from Junio remaining.

**Fuzz testing conflict in config migration** -- Olamide Caleb Bello's Outreachy project to migrate config`core.attributesFile` from global storage hit a snag when fuzz tests revealed initialization pattern mismatches with LibFuzzer's process model.

**Static build improvements** -- A patch series enabled static linking of Git binaries, particularly useful in container environments, by resolving symbol conflicts (notably around the "error" symbol) and adding `STATIC_CURL_LIBS` support.

**Documentation standardization** -- Kristoffer Haugsbakk had multiple documentation patches queued, including improvements to `git am` man pages and adding missing `am.messageId` config documentation, following the project's synopsis style conventions.

**Test modernization** -- Burak Kaan Karaçay and SoutrikDas sent patches converting legacy `test -f` checks to modern `test_path_is_*` helpers in several test files, part of the ongoing test infrastructure cleanup.

## On the radar

**Hook API modernization** -- Adrian Ratiu's configurable hooks series received detailed feedback from Patrick Steinhardt about memory management and string handling in the multi-hook infrastructure. The changes are foundational for both config-based hooks and parallel execution.

**ODB flag safety** -- Patrick Steinhardt's merged flag cleanup prompted follow-up discussion about compiler limitations in catching enum misuse, with Justin Tobler validating specific fixes in `builtin/backfill.c` and `builtin/fsck.c`.

**Gitweb mobile responsiveness** -- A new 5-patch series from Rito Rhymes aims to improve mobile rendering by adding viewport meta tags and mobile-specific CSS rules, with before/after screenshots showing the layout fixes.