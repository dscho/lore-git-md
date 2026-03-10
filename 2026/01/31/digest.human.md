# Git Mailing List Digest - 2026/01/31

**The day in brief.** A moderately busy Saturday with 25 emails across 11 threads, featuring progress on several ongoing efforts. The most notable developments include a finalized shallow clone fix, continued discussion about a proposed `@{default}` branch shorthand, and new submodule traversal options. Documentation and translation updates rounded out the day's activity.

## Notable threads

### Shallow clone boundary commit handling finalized

Samo Pogačnik's patch fixing edge cases in `--shallow-since` behavior has reached its third iteration, now resolving the last remaining terminology inconsistency ("border" vs "boundary") that Junio Hamano noted in previous reviews. The fix ensures proper handling of descendant relationships between boundary commits during shallow operations. With thorough test coverage and all prior feedback addressed, this appears ready for integration pending final review from shallow infrastructure experts.

### Debate continues on `@{default}` branch shorthand

The proposed `@{default}` branch reference syntax (now renamed to `@{primary}` in v4) has sparked extensive discussion about its conceptual foundations. Harald Nordgren's latest iteration pivots to resolve the primary branch through remote-tracking (`origin/HEAD`) rather than local configuration, better serving their use case of switching between projects with varying branch names. Junio Hamano questions whether this remote-centric approach aligns with Git's configuration model, suggesting `repo_default_branch_name()` might provide a better foundation. The thread has evolved from implementation details to fundamental questions about the feature's purpose and naming, with no clear resolution yet.

### Submodule foreach gains traversal options

A new five-patch series from Remy D. Farley enhances `git submodule foreach` with dependency-aware traversal capabilities. The additions include `--reverse-traversal` (processing children before parents), `--append-superproject` (including the top-level repository), and a `--reversive` shorthand combining both behaviors. The well-structured series includes comprehensive tests for the new options, which should prove useful for build systems and cleanup operations in complex submodule hierarchies. The implementation appears clean and focused, with documentation completed in the final patch.

## In brief

**HTTP authentication test coverage** -- A third version of tests for .netrc file handling now includes all requested documentation improvements from Junio Hamano, verifying behavior with 401 and 403 responses.

**Git 2.53.0-rc2 typo fix** -- Carlo Marcelo Arenas Belón corrected "did nt" to "did not" in the release notes for an upcoming submodule hash enforcement fix.

**Documentation synopsis style conversion** -- Jean-Noël Avila's v2 series converting several man pages to the new AsciiDoc format is ready for merging pending final review from Kristoffer Haugsbakk.

**Translation updates** -- Jiang Xin coordinated updates to Bulgarian, French, Irish, Indonesian, Swedish, Turkish, and Chinese translations for Git 2.53.0.

**CONTRIBUTING.md link fix** -- Abdalrhman Mohamed corrected a broken documentation link, sparking discussion about URL format consistency across the project's documentation.

**wt-status.c refactoring** -- A three-patch series continued the `the_repository` removal effort by eliminating global state from working tree status reporting.

**Pathspec exclusion fix** -- Remy D. Farley fixed an edge case where excluded pathspecs incorrectly respected `.gitignore` rules when using `git add :!x .`.

## On the radar

**`git add -p` navigation enhancements** -- The discussion about patch application timing (immediate vs transactional) continues, with Junio Hamano acknowledging potential value in both approaches while expressing personal preference for incremental checking.