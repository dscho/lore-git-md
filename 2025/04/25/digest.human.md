Here's the daily digest for April 25, 2025:

## The day in brief
April 25 saw significant activity across multiple fronts in Git development, with 142 emails across 20 threads. The day was dominated by technical refinements to ongoing series, including major progress on the bundle-uri performance optimizations, object store refactoring, and Meson build system improvements. Junio Hamano's "What's cooking" report provided a comprehensive snapshot of the current development landscape.

## Notable threads

**Bundle-uri reference negotiation reaches final polish**  
Scott Chacon's performance optimization series for bundle-uri reference handling reached its final iterations (v4 through v7) with all substantive feedback addressed. The series, now queued by Junio Hamano, demonstrates significant performance improvements (reducing object downloads from 32% to 1%) by expanding reference handling to all namespaces while preserving hierarchy. The final versions incorporated test improvements from Taylor Blau and documentation updates requested by Phillip Wood, with only minor whitespace fixes remaining in v7.

**Object store refactoring advances**  
Patrick Steinhardt's 13-part series refactoring the object store subsystem saw significant progress, with v2 incorporating all feedback on patches 3-9 from reviewer Karthik Nayak. The series systematically replaces `repo_has_object_file()` with a new `has_object()` API that provides explicit control over promisor object fetching. Key changes include improved index-pack collision checks and HTTP walker behavior modifications to avoid unnecessary promisor fetches. The series is part of the broader `the_repository` removal effort.

**Meson build system refinements**  
Multiple threads addressed Meson build system improvements, with particular focus on cross-platform reliability. A completed series standardized shell path detection to prefer `/bin/sh` while maintaining configurability, addressing issues seen in GitLab and Debian environments. Separate patches added runtime path reporting and improved platform-specific handling of functions like `getpagesize()` and networking libraries. The changes maintain functional equivalence with Makefile behavior while using Meson-native mechanisms.

**Windows-specific issues addressed**  
Several threads tackled Windows-specific concerns, including a resolution to Outlook SMTP Message-ID handling in `git send-email` (now queued as v6) and debugging guidance for lock file/reflog issues in Git for Windows. The CI system was also updated to handle Windows test hangs by building Meson tests in Release mode to avoid assertion dialogs.

## In brief

**Maintenance task decomposition** -- Derrick Stolee and Patrick Steinhardt completed a 7-patch series adding "worktree-prune" and "rerere-gc" maintenance tasks, marking the last components needed to fully replace `git gc` functionality.

**Xdiff minimal mode optimization** -- A performance patch modified xdiff's `cleanup_records` heuristic to produce shorter diffs in `--minimal` mode by skipping marking unchanged lines surrounded by changes.

**Perl library path customization** -- Đoàn Trần Công Danh added `perllibdir` configuration to Meson builds, matching Makefile flexibility for distributions with non-standard layouts.

**CI dependency handling** -- Patches made external dependency downloads (JGit, Perforce, Git-LFS) optional in CI scripts and switched JGit's source to Maven Central for improved reliability.

**Pack-objects bug report** -- Nikolas Garza reported and Elijah Newren confirmed a bug where `should_include_obj()` is called on non-existent objects, traced to a v2.48.0 change in promisor pack handling.

## On the radar

**Blame ignore-revs usability** -- Michael Grosser proposed making `git blame` more forgiving of missing `.git-blame-ignore-revs` files when using global configuration, with Eric Sunshine surfacing relevant prior discussion.

**Authen-SASL distribution** -- With OAuth2 support now released in Authen-SASL 2.1800, attention turns to when major Linux distributions will package the update, as it's a dependency for `send-email` OAuth2 functionality.

**Windows case sensitivity issues** -- A bug report detailed persistent case-sensitivity problems on Windows involving phantom untracked files that resist standard recovery procedures, suggesting deeper filesystem interaction issues.