# Git Mailing List Digest - 2026/01/31

**The day in brief.** A moderately busy Saturday with 25 emails across 11 threads, featuring progress on several ongoing efforts. Notable items include a finalized fix for shallow clone edge cases, continued discussion about a proposed `@{default}` branch shorthand, and a new submodule traversal feature series. The `the_repository` removal effort saw another incremental refactoring, while documentation and translation updates rounded out the day's activity.

## Notable threads

### Shallow clone boundary commit fix finalized

Samo Pogačnik's patch fixing edge cases in `--shallow-since` behavior has reached its third iteration, now resolving the last remaining terminology inconsistency ("border" vs "boundary") that Junio C Hamano noted in previous reviews. The fix addresses incorrect handling of descendant relationships between boundary commits during shallow clones by implementing a more sophisticated painting algorithm in `shallow.c`. With thorough test coverage and all prior feedback addressed, this appears ready for integration pending final review from shallow infrastructure experts. The two-month discussion has refined both the implementation and its documentation, demonstrating Git's careful approach to subtle behavior changes.

### Debate continues on `@{default}` branch shorthand

Harald Nordgren's proposal for a `@{default}` shorthand to reference a repository's primary branch has evolved through four iterations, with the latest version pivoting to resolve the branch via remote-tracking (`origin/HEAD`) rather than local configuration. Junio Hamano raised fundamental concerns about the naming and conceptual alignment, questioning whether "default" accurately describes remote-tracking behavior and suggesting `repo_default_branch_name()` might be a better foundation. Nordgren provided concrete workflow examples showing the utility across repositories with varying branch names (main/master/trunk), but the discussion remains unresolved on whether the shorthand should represent configured defaults or actual usage patterns. The thread has shifted from implementation details to first-principles questions about the feature's purpose.

### Submodule foreach gains traversal options

A new five-patch series from Remy D. Farley enhances `git submodule foreach` with dependency-aware traversal options. The additions include `--reverse-traversal` (processing children before parents), `--append-superproject` (including the top-level repository), and a `--reversive` shorthand combining both behaviors. The well-structured series introduces comprehensive test coverage first, then implements each feature incrementally before concluding with documentation updates. These options address real needs in complex submodule hierarchies where operations like builds or cleanups require specific ordering. The changes are focused and appear ready for review, following Git's standard pattern for adding functionality without disrupting existing behavior.

### `the_repository` removal progresses in wt-status

René Scharfe continued the project-wide effort to eliminate the `the_repository` global variable, submitting a three-patch series refactoring `wt-status.c`. The changes follow the established pattern: first substituting local repository instances for the global, then updating function signatures to explicitly pass repository state, and finally removing `the_hash_algo` dependencies. The mechanical but thorough modifications (totaling 76 lines changed) demonstrate Git's incremental approach to architectural changes, preserving behavior while making dependencies explicit. This series is part of a multi-year effort touching nearly every subsystem.

## In brief

**Shallow clone boundary fix** -- Samo Pogačnik's v3 patch fixes edge cases in `--shallow-since` behavior, now consistently using "boundary" terminology throughout as requested in review.

**HTTP auth .netrc tests** -- Third version of a test patch verifying .netrc credential handling with 401/403 responses, incorporating all prior review feedback about test conventions.

**Git 2.53.0-rc2 typo fix** -- Carlo Marcelo Arenas Belón corrected "did nt" to "did not" in the release notes for a submodule hash enforcement fix.

**Documentation link fix** -- Abdalrhman Mohamed fixed a broken relative link in CONTRIBUTING.md, sparking discussion about URL format consistency across Git's documentation.

**L10n updates** -- Jiang Xin coordinated translation pulls for 8 languages (Bulgarian, French, Irish, Indonesian, Swedish, Turkish, Chinese) totaling over 7,000 lines of updates.

**Pathspec exclusion fix** -- Remy D. Farley fixed an edge case where excluded pathspecs (`:!x`) incorrectly respected `.gitignore` rules for commands like `git add` and `git stash`.

## On the radar

**`git add -p` navigation** -- The discussion about when to apply patch decisions (immediately vs transactionally) continues, with Junio acknowledging both approaches have merit but suggesting configurability.