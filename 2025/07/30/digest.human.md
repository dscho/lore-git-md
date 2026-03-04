# Git Mailing List Digest - 2025/07/30

**The day in brief.** A moderately active day with 53 emails across 19 threads, featuring significant progress on several fronts. The `git last-modified` command series reached maturity in its sixth iteration, while architectural discussions continued around sparse-checkout configuration and reflog behavior. Platform-specific work dominated the latter half of the day with extensive discussion about git-gui's macOS support.

## Notable threads

### `git last-modified` reaches v6

The long-running `git last-modified` series reached its sixth iteration, consolidating core functionality for tracking tree-level path modifications. The v6 version drops controversial formatting options in favor of focused forge-style use cases, adds Bloom filter optimizations (50.6% speedup for top-level checks), and includes comprehensive tests and benchmarks. While recursive scenarios still show room for improvement (only 0.4% speedup), the series appears ready for merging with its 300-line core implementation and thorough test coverage.

### Sparse-checkout config refactoring debate

Phillip Wood proposed an alternative approach to handling sparse-checkout configurations that would modify `git_default_config()` to accept a repository pointer, avoiding globals while maintaining "last one wins" behavior. Junio Hamano acknowledged the approach would be "very painful" to implement but conceptually worthwhile, keeping this architectural option in play alongside Ayush Chandekar's more incremental `struct repository` work.

### Reflog DWIM behavior discussion

A nuanced debate emerged about whether the new `git reflog write` subcommand should use DWIM (Do What I Mean) behavior for refname resolution like other reflog operations. Patrick Steinhardt and Junio Hamano agreed the plumbing-like command should require fully-qualified refnames, with Junio musing that broader reflog DWIM behavior might have been a historical UI mistake. Karthik Nayak later investigated the inconsistency between strict `write` and DWIM behavior in other reflog operations, with Junio suggesting either full consistency or clear documentation of the difference.

### `git status --json` proposal faces scrutiny

Tachera Sasi's proposal to add JSON output to `git status` received rigorous technical review. Phillip Wood identified several robustness issues (filename encoding, JSON spec compliance), while Junio Hamano questioned the fundamental need given existing porcelain formats. The discussion revealed deeper questions about status formatting architecture, with Junio suggesting preliminary refactoring of format selection before adding new output options.

## In brief

**SMTP autoconfiguration for send-email** -- Aditya Garg introduced a `--get-smtp-server` option that automatically discovers settings via Mozilla's ISPDB, provider endpoints, and MX lookups, now in its second iteration.

**MIDX refactoring series polish** -- Patrick Steinhardt corrected a typo in his series title ("stop duplicating" instead of "deduplicating") for the multi-pack index changes that are otherwise ready for merging.

**Test path correction** -- Chen Jianhu fixed a submodule test path in t7450-bad-git-dotfiles.sh, with follow-up discussion about proper author name formatting and commit message clarity.

**Depth calculation fix** -- Toon Claes corrected edge cases in `within_depth()` handling for the upcoming `--max-depth` diff feature, with Patrick Steinhardt reviewing the changes.

**Help system improvements** -- D. Ben Knoble's series expanding `-h` and `--help-all` behavior saw patch 3 approved while the controversial `-h` mid-command functionality remains debated.

## On the radar

**git-gui macOS modernization** -- A 9-part series removing deprecated Tcl/Tk 8.5 support sparked deep architectural discussion about whether Tcl/Tk should be treated as a system component or updatable dependency on macOS, with Carlo Arenas providing key context about the platform's integration challenges.