# Git Mailing List Digest - 2025/07/30

**The day in brief.** A moderately active day with 53 emails across 19 threads, featuring significant progress on several fronts. The `git last-modified` command series reached v6 with Bloom filter optimizations, while architectural discussions continued around sparse-checkout config handling and reflog DWIM behavior. Notable new features included JSON output for `git status` and SMTP autoconfiguration for `git send-email`.

## Notable threads

**`git last-modified` reaches v6 with Bloom filters**  
The long-running series implementing a new `git last-modified` command for tracking tree-level path modifications (distinct from line-by-line `git blame`) has reached v6 iteration with significant refinements. The latest version consolidates the implementation after extensive discussion and benchmarking, focusing on core functionality while deferring some optimizations. Key changes include removing the `--format` option (deferred to future work), exporting `prepare_commit_graph()` for Bloom filter integration, and simplifying the callback architecture. Performance remains mixed - 55% speedup for top-level checks but only 0.4% improvement in recursive scenarios. The series now includes comprehensive tests and performance benchmarks, with subdirectory test selection improved via `git ls-tree -d`. The implementation spans about 300 lines of core logic with thorough test coverage, positioned as a focused solution for forge view use cases.

**Sparse-checkout config refactoring debate continues**  
Phillip Wood proposed an alternative approach to handling sparse-checkout configurations that would modify `git_default_config()` to accept a repository pointer as callback data. This attempts to address both the global variable elimination goal and late config parsing concerns while maintaining "last one wins" behavior. Junio Hamano acknowledged the approach would be "very painful" to implement due to extensive callback chain modifications but agreed it seems conceptually worthwhile. The discussion continues to explore architectural tradeoffs between invasiveness and cleanliness in eliminating sparse-checkout globals.

**`git status --json proposal faces review scrutiny**  
A new feature patch adding JSON output format to `git status` received rigorous review feedback. Phillip Wood identified several technical concerns including improper handling of filenames with special characters, UTF-8 encoding assumptions, trailing commas in JSON arrays, and lack of tests. Junio Hamano later questioned whether JSON output is truly needed given existing porcelain formats, suggesting preliminary refactoring of format selection infrastructure should come first. The discussion revealed differing perspectives on whether the new format provides sufficient value over existing machine-readable options, with the implementation likely needing significant refinement before progressing.

**SMTP autoconfiguration for git-send-email**  
A feature patch introduced a `--get-smtp-server` option to `git send-email` that automatically discovers SMTP settings through three fallback methods: querying Mozilla's ISPDB, checking provider autoconfig endpoints, and MX record lookup for custom domains. The implementation adds 157 lines to git-send-email.perl and requires new Perl modules (Net::DNS, URI::Escape, XML::LibXML). The v2 revision improves email address checking and models its behavior after Thunderbird's autoconfig system, providing structured output that maps to existing sendemail.* config options. While promising for simplifying setup, the new dependencies may prompt discussion about whether they're too heavy for core Git.

**git-gui macOS modernization sparks architectural debate**  
A series modernizing git-gui's macOS support by removing deprecated components prompted fundamental questions from Junio Hamano about whether Tcl/Tk should be treated as an immutable system component or updatable third-party dependency. Carlo Arenas clarified that while Tcl/Tk is system-integrated on macOS as a Framework (and outdated at version 8.5), the core issue is git-gui's automatic detection behavior favoring this incompatible system version. The discussion revealed tensions between platform-specific fixes and general build system portability, with several technical issues (RUNTIME_PREFIX breakage, Meson compatibility) needing resolution before the series can progress.

## In brief

**Reftable write command refname validation** -- Patrick Steinhardt and Junio Hamano confirmed `git reflog write` will require fully-qualified refnames without DWIM behavior, comparing it to `update-ref`'s plumbing-like design.

**`-I<regex>` diff output inconsistency** -- Junio Hamano suggested a fix for inconsistent `-I<regex>` behavior across diff formats should address the broader class of `diff_from_contents` cases rather than just regex matching.

**`git grep` pathspec parsing fix** -- D. Ben Knoble identified the root cause of `git grep` failing to handle `:^:` pathspec syntax, tracing it to `verify_filename()` not properly handling the optional trailing colon in shorthand magic pathspecs.

**Help system behavior refinements** -- D. Ben Knoble's series expanding `-h` and `--help-all` functionality saw patch 3 (enabling `--help-all` outside repositories) approved while patch 4 (allowing `-h` mid-command) remained controversial due to security concerns.

**MIDX refactoring title correction** -- Patrick Steinhardt acknowledged a typo in his MIDX refactoring series title ("stop deduplicating info" should say "stop duplicating info"), marking the final polish before potential merging.

**Test fix for submodule checkout paths** -- A correction to t7450-bad-git-dotfiles.sh verified proper path checking during submodule checkouts, with Justin Tobler confirming it accurately tests the security fix from commit 05e9cd64.

## On the radar

**`the_repository` removal effort** -- Phillip Wood reinforced that mechanical conversions must preserve existing config precedence rules, even for deprecated features, as seen in the `fmt-merge-msg` discussion about `merge.log` vs `merge.summary`.

**Depth-limited diffs** -- Patrick Steinhardt's review of Toon Claes' `--max-depth` diff feature identified several implementation improvements needed around error handling, option parsing, and type safety.

**Rustification discussions** -- While not active today, Ezekiel Newren's effort to introduce Rust code into Git remains a contentious topic with Randall S. Becker's platform support concerns still unresolved.