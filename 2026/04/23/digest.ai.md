Here's the daily digest for April 23, 2026:

## The day in brief

A moderately busy day with 72 emails across 15 threads, featuring several patch series reaching maturity and a notable regression report. Key developments include the finalization of subcommand autocorrection, progress on the `the_repository` removal effort, and a new `git history fixup` command. A regression in hierarchical alias handling emerged as the most pressing issue needing attention.

## Notable threads

**Subcommand autocorrection finalized**  
Jiamu Sun's subcommand autocorrection series reached its sixth and likely final iteration, addressing test failures that surfaced during final review. The feature provides consistent autocorrection behavior whether users mistype main commands or subcommands, with configurable interaction modes. Junio Hamano identified test failures in t0040 and t7900, which Sun promptly fixed by updating test expectations and standardizing exit codes to 129. The series now has all components implemented, tested, and documented with maintainer approval, making it ready for merging.

**`the_repository` removal continues**  
Olamide Caleb Bello's v3 series migrated eight configuration variables into `struct repo_config_values` as part of the ongoing effort to eliminate the `the_repository` global. The changes affect file status checking, compression settings, Unicode handling, and sparse-checkout behavior while maintaining eager parsing semantics. The series received positive reviews from Karthik Nayak and Patrick Steinhardt, with only minor documentation polish remaining. A parallel thread saw Ramsay Jones fix a sparse warning in parse-options.c related to recent subcommand autocorrection changes.

**`git history fixup` command introduced**  
Patrick Steinhardt proposed a new `git history fixup` subcommand (v2) that amends staged changes to an existing commit while automatically rebasing dependent branches. The series grew to three patches with improved empty commit handling via a new `--empty` flag, addressing feedback from Tian Yuchen about file deletion edge cases. Documentation discussions continue, with D. Ben Knoble advocating for user-focused explanations over technical details. The implementation uses merge-ort for three-way merging and includes extensive test coverage.

**Hierarchical aliases regression**  
Michael Grossfeld reported a regression where hierarchical aliases (like `pull.sub`) stopped working in Git 2.54.0. Jeff King and René Scharfe confirmed the issue stems from commit ac1f12a9de introducing new alias syntax, with Scharfe proposing a solution that would restore most broken aliases while maintaining new functionality. The thread converged on a fix path that addresses real-world usage while acknowledging a minor backward compatibility tradeoff for aliases ending in `.command`.

## In brief

**Reftable revision walk option** -- Mirko Faina's v3 series adds `--reverse=before` for performing commit reversal before limiting, with new memory optimizations and finalized documentation incorporating Jeff King's suggestions.

**Ref backend consolidation** -- Karthik Nayak's v3 series centralizes object validation and peeling logic in the refs layer, with Patrick Steinhardt approving the reflog configuration changes as "a ton easier to review."

**Git grep column numbers** -- Phillip Wood joined the discussion about incorrect column reporting with `--only-matching`, supporting René Scharfe's proposal to match GNU grep's absolute positioning.

**Push progress configuration** -- A new patch adds `push.showProgress` to control progress reporting during push operations, allowing persistent configuration of the verbose output.

**CI workflow updates** -- Christoph Grüninger proposed GitHub Actions version bumps to address Node.js 20 deprecation warnings, following up on earlier Dependabot updates.

## On the radar

**Worktree-specific gitignore** -- D. Ben Knoble and brian m. carlson discussed making `info/exclude` files worktree-specific, with a proposed solution using the extensions mechanism to maintain backward compatibility.