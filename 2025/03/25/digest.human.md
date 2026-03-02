Here's the daily digest for March 25, 2025:

## The day in brief
A moderately busy day with 86 emails across 18 threads, featuring significant discussions around release coordination between Git core and Git for Windows, continued Perl test suite modernization, and a systematic cleanup of comma operator usage in the codebase. The Git Merge 2025 venue discussion also saw concrete proposals for Canadian alternatives to the planned San Francisco location.

## Notable threads

**Release schedule coordination between Git core and Git for Windows**  
Johannes Schindardt provided extensive references advocating against Friday releases (citing industry practices from curl, Firefox, and Microsoft's Patch Tuesday), though Junio Hamano pressed for a concrete proposal rather than indirect arguments. The discussion revealed differing communication styles while keeping the door open for alignment between Git core's Friday releases and Git for Windows' Monday schedule. The technical resolution for the original Windows lockup issue remains unchanged (fixed in upcoming Git for Windows v2.49.0).

**Perl test suite modernization reaches milestone**  
Patrick Steinhardt's 20-part series to make Perl optional in Git's test suite saw its second version posted, now achieving 97% test coverage (30342/31358 tests) without Perl. The series systematically replaces Perl-based test helpers with shell/awk/sed implementations while introducing PERL_TEST_HELPERS for tests that still require Perl. Notable conversions include environment sanitization, textconv script processing, and trace2 log scrubbing. Reviewers Eric Sunshine and Phillip Wood validated that test behavior remains intact despite implementation changes.

**Systematic comma operator cleanup**  
What began as a simple two-patch series from Johannes Schindelin to remove unnecessary comma operators grew into a comprehensive 10-part refactoring effort after Clang's -Wcomma flag revealed additional cases. The series now spans multiple subsystems (remote-curl, rebase, kwset, xdiff) with a balanced approach: removing questionable comma operators while preserving performance-critical uses in diff algorithms and regex code. The final patches add Clang warning infrastructure to prevent future issues. Junio Hamano approved the initial patches, calling the original comma usage "suspicious enough that one might suspect malicious intent."

**Git Merge 2025 venue alternatives emerge**  
Following concerns about US accessibility, Randall Becker provided detailed Toronto venue options (Metro Toronto Convention Centre, Westin Harbour Castle) with capacity for 1500+ attendees, while Brian M. Carlson offered local organizational support. This builds on Patrick Steinhardt's earlier Amsterdam proposal (motivated by OSS Summit colocation). The discussion has shifted from whether Canada could work to practical implementation details, though the official announcement later confirmed San Francisco as the location while acknowledging accessibility concerns.

## In brief

Justin Tobler's v3 series fixing advice message suppression in clone operations received positive review from Phillip Wood, addressing all edge cases including bundle clone scenarios. The SMTP error handling refactoring reached final pre-merge review with Junio suggesting minor code organization improvements. A Windows compatibility patch series addressed administrator file ownership edge cases in safe directory checks, adding manual debugging tools since automated testing of elevated scenarios isn't possible. The Rust crate packaging discussion explored alternatives to symlinks for Windows compatibility, with Junio open to root-level Cargo.toml if it maintains project cleanliness.

## On the radar

The git-blame porcelain output discussion converged on restricting ignored/unblamable status markers to --line-porcelain mode, with Junio confirming this properly accommodates per-line metadata. The Git Merge venue discussion continues to surface accessibility concerns despite the official San Francisco announcement, suggesting this topic may resurface in future planning. The comma operator cleanup has expanded beyond its original scope into a systematic codebase audit, with more patches likely as additional cases are identified via Clang warnings.