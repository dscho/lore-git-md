# Git Mailing List Digest - 2025/09/17

**The day in brief.** A busy Wednesday with 130 emails across 29 threads, dominated by Rust infrastructure discussions, batched reference updates, and documentation improvements. Key developments include final approval for the case-insensitive filesystem fixes, ongoing coordination of Rust integration efforts, and significant progress on command deprecation and signature handling features.

## Notable threads

### Rust infrastructure coordination

The Rust integration effort saw extensive discussion today as Ezekiel Newren and Patrick Steinhardt work to align their parallel patch series. Junio Hamano raised concerns about duplicate commits and conflicting version numbering between the two approaches, prompting Elijah Newren to propose four potential resolution paths. The discussion revealed substantive differences in rollout timelines (Steinhardt favors a slower adoption) and technical approaches to cargo delegation and type definitions. Meanwhile, Eric Sunshine provided detailed build system reviews focusing on style consistency and documentation clarity, while platform-specific concerns emerged from distribution maintainers about dependency management and offline builds.

### Batched reference updates finalized

Karthik Nayak's v4 series fixing case-insensitive filesystem issues in batched reference updates received final approvals from Justin Tobler, Toon Claes, and Junio Hamano. The comprehensive solution handles four conflict types (direct case collisions, file/directory conflicts, directory/file conflicts, and lock file problems) through new error codes (`REF_TRANSACTION_ERROR_CASE_CONFLICT`, `REF_TRANSACTION_ERROR_NAME_CONFLICT`) and careful lock file management in the files backend. While a theoretical edge case regarding lock` file naming conflicts remains, the series is now cleared for merging after four iterations of review.

### Command deprecation framework

Kristoffer Haugsbakk's v6 series implementing Git's command deprecation system is now complete with build system fixes for `git-whatchanged`. The series enhances user experience with actionable error messages suggesting alternatives to deprecated commands, while maintaining backward compatibility through alias shadowing support. The final patches address a Makefile inconsistency where `git-whatchanged` remained in `BUILT_INS` when built with `WITH_BREAKING_CHANGES`, ensuring proper "not a git command" errors rather than confusing builtin handling messages.

### Signature handling in fast-import

Christian Couder's v3 series adding signature handling to `git fast-import` reached its final form with improved code organization and robustness. The implementation now splits signature parsing into dedicated helper functions (`import_one_signature()` and `discard_one_signature()`) and includes comprehensive test coverage of all signature formats (OpenPGP/X.509/SSH) and handling modes. While higher-level design questions remain about mode naming conventions, the technical implementation appears solid and ready for merging.

## In brief

**Shallow clone limitations** -- Elijah Newren clarifies that commit hash-based shallow exclusion is considered "flawed" in the current implementation due to technical constraints rather than fundamental protocol issues.

**Pull shorthand proposal** -- Julien Jerphanion justifies adding `-u` as shorthand for `--set-upstream` in `git pull` by citing fork-based workflow needs in projects like conda-forge.

**Documentation improvements** -- Julia Evans' `git-checkout` documentation series and subsequent `git-push` man page updates received final approval, with Junio Hamano endorsing the clearer organization and terminology choices.

**Default branch transition** -- Junio Hamano finalized the advice messaging strategy for Git's switch from "master" to "main" as the default branch name when built with `WITH_BREAKING_CHANGES`.

**String-list refactoring** -- A completed series modernized the string-list API to use proper types (`size_t` for indices, `bool` for flags), eliminating sign comparison warnings while maintaining identical behavior.

**Bundle-URI error handling** -- Toon Claes and Justin Tobler continued debating whether client-side workarounds or server-side fixes are preferable for handling invalid bundle-URI responses, with Claes emphasizing real-world broken server scenarios.

**Worktree safety checks** -- Gabriel Scherer proposed a new `--detach-other-worktrees` option as a middle ground between strict safety checks and permissive `--ignore-other-worktrees` behavior for branch conflicts.

**Send-email header fix** -- NeilBrown addressed a `git send-email` bug where using both `--compose` and `--reply-to` created duplicate Reply-To headers, with the fix now ready for merging after editorial polish.

## On the radar

**Rust style guidelines** -- An unresolved discussion continues about whether to follow Rust's conventional 100-column limit or maintain Git's traditional 80-column standard via project-wide `rustfmt.toml` configuration.

**SHA-1/SHA-256 interop** -- SZEDER Gábor raised concerns about how the Rust transition might affect SHA-1/SHA-256 interoperability features if some platforms remain on the LTS release without Rust support.

**Dry-run filter behavior** -- A bug report revealed `git fetch --dry-run --filter` unexpectedly modifies local config by adding promisor remote settings, violating the dry-run contract while highlighting limitations in commit existence checking.