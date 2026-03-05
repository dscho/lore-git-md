# Git Mailing List Digest — 2025/08/09

**The day in brief.** A moderately active Saturday with 22 emails across 11 threads saw the conclusion of several significant efforts. Lidong Yan's Bloom filter optimization for wildcard pathspecs reached its final form, Julia Evans' `git-rebase` documentation overhaul was completed, and Junio Hamano provided a comprehensive "What's cooking" status update. The day's technical work focused on performance optimizations, documentation refinements, and platform-specific fixes.

## Notable threads

**Bloom filters now handle wildcard pathspecs** — Lidong Yan's performance optimization series reached its v4 iteration, enabling Bloom filters to work with wildcard pathspecs by matching only the non-wildcard directory portion. The implementation introduces an `allowed_magic` mask to selectively enable compatible pathspec signatures (top, glob, attr, depth, literal) while excluding others. Benchmarks show 18-27% speedups in Git's test suite and 47-63% improvements in larger repositories like LLVM. Junio Hamano provided final code quality feedback about error handling patterns, though the core logic was already technically sound with comprehensive test coverage in t4216-log-bloom.sh.

**git-rebase documentation overhaul complete** — First-time contributor Julia Evans successfully landed her five-patch series restructuring the `git-rebase` man page to improve accessibility while preserving all technical content. The changes front-load practical examples, consolidate duplicate explanations (particularly around conflict resolution), and better organize advanced topics like `--onto` usage into dedicated sections. The series went through four iterations with maintainer feedback, finalizing terminology choices (standardizing on `git checkout` rather than `git switch` in examples) and adding cross-references. D. Ben Knoble later suggested an additional cross-reference to improve discoverability of the `--onto` examples.

**Heap overflow fix verified for git diff --no-index** — Gregoire Geis confirmed that Junio Hamano's architectural solution for a buffer overflow in `git diff --no-index` when comparing stdin in subdirectories works as intended. The fix properly handles path comparisons while maintaining expected behavior, with ASAN no longer reporting overflows and the test suite passing. This concludes the thread addressing a subtle memory safety issue in diff's no-index mode.

## In brief

**What's cooking report** — Junio Hamano's status update noted several bugfixes graduating to 'master', ongoing `the_repository` removal work (17 commits in ps/object-file-wo-the-repository), and new features like `git refs list` and IMAP sent folder support in development.

**for-each-ref documentation consistency** — René Scharfe standardized terminology for the `--start-after` option's argument, changing the short help text from "start-after" to "marker" to match other documentation.

**merge --compact-summary fix** — René Scharfe corrected an incorrect help string that misleadingly suggested the no-argument `--compact-summary` option took a parameter.

**Cygwin test prerequisite formatting** — Ramsay Jones and Junio Hamano refined a skip message in t6137-pathspec-wildcards-literal.sh to match test suite conventions ("missing BSLASHPSPEC" rather than "skipping: needs BSLASHPSPEC").

**git-jump filename handling** — A confirmed fix for spaces in filenames dropped support for the edge case of tab-terminated names, with potential future solutions discussed if needed.

**diff --no-index build fix** — Ramsay Jones reported and provided a quick fix for a DEVELOPER=1 build failure in the 'seen' branch due to unchecked `chdir()` return values.

## On the radar

**subtree split failure report** — A new bug report describes `git subtree split` failing with "no new revisions were found" when processing certain merge commits, with reproduction steps provided but no root cause analysis yet.