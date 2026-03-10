Here's the Git mailing list digest for February 25, 2026:

## The day in brief

A busy Wednesday with 121 emails across 28 threads saw several major features reach completion while new proposals emerged. The reference backend selection series concluded after nine iterations, configurable branch comparisons for `git status` were approved after 28 versions, and Linux fsmonitor support stabilized. Meanwhile, Taylor Blau proposed a significant MIDX/bitmap repacking strategy and discussions around `git log` decoration behavior heated up.

## Notable threads

**Reference backend selection finalized**  
After nine-iteration journey, Karthik Nayak's series implementing zero-downtime migrations between reference backends (files<->reftable) received final approval from both Patrick Steinhardt and Junio Hamano. The implementation provides three configuration mechanisms (`extensions.refStorage` config, `GIT_REFERENCE_BACKEND` environment variable, and URI syntax) with proper precedence rules and stub management. Junio performed final style cleanups before merging to 'next', marking completion of this feature primarily serving GitLab's migration needs.

**Configurable branch comparisons approved**  
Harald Nordgren's 28-iteration effort to add `status.compareBranches` configuration reached conclusion after being simplified to focus exclusively on `@{upstream}` and `@{push}` comparisons. The thread became a case study in Git's review culture, with Jeff King and Junio Hamano guiding the first-time contributor through the mailing list workflow. The final version includes comprehensive test coverage (337 test lines) and clear documentation of its intentional limitations, with maintainer confirmation that all substantive concerns were addressed.

**Linux fsmonitor stabilizes**  
Paul Tarjan's Linux filesystem monitoring implementation using inotify reached its sixth iteration with all major technical issues resolved. The series brings Linux to parity with existing Windows and macOS backends, featuring recursive directory watching, rename handling via event cookies, and remote filesystem detection. While minor memory leaks remain (512-byte during daemon init, 40-byte in IPC handling), these are considered non-blocking. The implementation has been stable in production for two months and is now in Junio's 'seen' branch pending final CI test fixes from Patrick Steinhardt.

**MIDX/bitmap repacking proposed**  
Taylor Blau introduced an RFC for incremental MIDX/bitmap-based repacking, building on his previous MIDX compaction work. The 14-patch series proposes avoiding periodic all-into-one repacks through geometric repacking of non-MIDX'd packs while maintaining MIDX layer structure. New features include `--checksum-only` and `--base` MIDX options, configurable layer thresholds, and compaction heuristics. The substantial architectural change could significantly impact large repository performance and is expected to generate extensive discussion from pack/MIDX experts.

**Decoration behavior debate**  
A lively discussion emerged around `git log`'s, specifically whether the `%d` format specifier should respect `--decorate` options. Alejandro Colomar's initial proposal to make `%d` conditional led to Junio Hamano suggesting a new `%(decorate:optional=yes)` placeholder, while Marc Branchaud argued for extending `--decorate=no` to suppress all decorations. The thread revealed current inconsistencies in documentation versus implementation and examined real-world usage patterns, with participants carefully weighing backward compatibility against interface coherence. The discussion remains ongoing but productive.

## In brief

**Pre-add hook v3** -- Chandra Kethi-Reddy's `pre-add` hook for validating staged changes advanced to v3 with documentation formatting as the final open item. The implementation provides temporary files representing original and proposed index states.

**Worktree API refinements** -- Phillip Wood and Junio Hamano discussed architectural questions about worktree-repository relationships that emerged from API simplifications, particularly around `is_current_worktree()` behavior in multi-repository contexts.

**Submodule remote fix** -- A bugfix addressing submodule fetching's assumption that all remotes are named "origin" was finalized, adding a `get-default-remote` helper command with comprehensive test coverage.

**Maintenance list command** -- Rémy Léone contributed a `git maintenance list` subcommand to display repositories registered for background maintenance, reading from `maintenance.repo` config.

**Config-list.h dependencies** -- D. Ben Knoble's build system patch ensuring `config-list.h` stays synchronized with documentation files was approved after addressing a file removal edge case in v7.

## On the radar

**Parallel hooks progress** -- Adrian Ratiu's config-based hooks series (v3) is now merge-ready while parallel execution improvements proceed as a separate track, with email delivery issues between Adrian and brian m. carlson resolved.

**GSoC proposals** -- Multiple GSoC draft proposals emerged, including Tian Yuchen's `the_repository` reduction work and Amisha Chhajed's partial clone disk reclamation idea, with Derrick Stolee providing guidance on technical direction.

**Cover letter formatting** -- Mirko Faina's configurable cover letter format series saw extensive review from Jeff King, who proposed extending format strings to customize patch numbering prefixes while questioning their necessity.