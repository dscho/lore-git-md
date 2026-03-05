# Git Mailing List Digest - 2025/08/08

**The day in brief.** A moderately busy Friday with 67 emails across 14 threads saw several long-running efforts reach completion, including the `git repo info` command series and a major documentation reorganization of `git-rebase`. Notable progress was made on performance optimizations for Bloom filters with wildcard pathspecs, while platform-specific test fixes and documentation improvements rounded out the day's activity.

## Notable threads

### `git repo info` command approved for merging

The `git repo info` command series from Lucas Seiki Oshiro received final approvals from both mentors (Patrick Steinhardt and Karthik Nayak) and was marked ready for inclusion in 'next' by Junio Hamano. This GSoC project introduces a structured way to query repository metadata (reference format, bare/shallow status) as an alternative to `git rev-parse` usage. The v9 iteration addressed final style refinements including test message standardization and BUG() formatting. The implementation uses a dual strbuf architecture with bsearch field lookup and supports both human-readable and machine-readable output formats.

### Rebase documentation reorganized for clarity

Julia Evans completed a 5-patch series restructuring the `git-rebase` man page to improve accessibility for newcomers while preserving all technical content. The changes front-load basic examples, consolidate merge conflict resolution instructions, clarify argument syntax, and move detailed `--onto` explanations to a dedicated section. Maintainer Junio Hamano provided extensive feedback throughout the series, which was incorporated in the final v3 version that has now been merged. The reorganization mirrors the approach used in `git-merge` documentation and represents a significant improvement in making rebase's complex functionality more approachable.

### Bloom filter optimization for wildcard pathspecs

Lidong Yan's v2 patch to enable Bloom filter usage with wildcard pathspecs showed promising performance improvements (18-27% faster in Git repo, 47-63% in LLVM repo). The implementation now uses an allow-mask approach for pathspec magic handling (renamed to `allowed_magic`) and falls back to non-filtered traversal when wildcards cover entire paths. Junio Hamano reviewed the edge case handling, confirming correct behavior for mixed wildcard/non-wildcard pathspecs while suggesting code simplifications for the zero-length path case. The comprehensive test suite in t4216-log-bloom.sh verifies both positive and negative cases.

### Diff no-index stdin path fix validated

Gregoire Geis confirmed Junio Hamano's architectural fix for the `git diff --no-index` stdin path issue works in their environment. The solution properly separates repository assumptions from no-index mode by clearing Git's startup prefix during operation, resolving both the immediate heap overflow and the underlying architectural mismatch. The minimal 14-line change in `builtin/diff.c` maintains the expected stdin output format ("a/- b/-") while fixing the memory safety issue. Test coverage was added in t4053-diff-no-index.sh.

### Test reorganization completed

Usman Akinyemi's v6 series finalizing the test strategy for help output verification was approved for merging into 'next'. The changes complete the migration of repository-dependent help tests from t1517-outside-repo.sh to command-specific test files (t5200 for update-server-info, t5304 for prune) while standardizing the outside-repo testing approach using `git --list-cmds=main`. Windows-specific handling of the `instaweb` test via PERL prerequisites was the final technical refinement needed before approval.

## In brief

**Content-ignoring diff options** -- Lidong Yan's v5 patch finalizes the fix for inconsistent behavior between `-w`/`-I` and metadata outputs like `--name-status`, using a new `dry_run` mode in the diff machinery.

**Push error handling improvements** -- Denton Liu's series replacing a BUG() with advise() when pushing invalid OIDs to unqualified refs was approved after addressing all feedback, including a switch statement refactoring.

**Documentation linting tool** -- Jean-Noël Avila added a Perl linter to enforce proper definition list formatting in man pages, fixing 4 documentation files as part of ongoing style standardization.

**Cygwin test fix** -- Ramsay Jones corrected a pathspec test's prerequisite from FUNNYNAMES to BSLASHPSPEC for proper Windows/Cygwin compatibility, with follow-up formatting improvements to the skip message.

**git-jump space handling** -- A one-line Perl regex fix addresses `git-jump`'s handling of filenames with spaces in diff output by stripping trailing tabs that were causing Vim quickfix failures.

**Documentation formatting fix** -- A simple correction fixed `log.decorate` config options being incorrectly formatted as a literal block instead of a description list in the manual.