# Git Mailing List Digest - 2025/08/09

**The day in brief.** A moderately active Saturday with 22 emails across 11 threads saw the conclusion of several significant efforts. The Bloom filter optimization for wildcard pathspecs reached its final form, Julia Evans' `git-rebase` documentation overhaul completed merging, and Junio Hamano provided a comprehensive "What's cooking" status update. While no major controversies emerged, the day was marked by steady progress on multiple fronts.

## Notable threads

**Bloom filters now handle wildcard pathspecs** -- Lidong Yan's performance optimization series reached its v4 iteration, incorporating final feedback from Junio Hamano. The patch enables Bloom filters to work with certain wildcard pathspecs by using only the non-wildcard portion up to the last directory separator. Benchmarks show 18-27% speedups in Git's own repository and 47-63% improvements in larger projects like LLVM. The implementation carefully handles edge cases through an `allowed_magic` bitmask that permits compatible pathspec signatures (top, glob, attr, --depth, literal) while excluding others. After multiple rounds of review addressing memory management and error handling, the patch appears ready for inclusion.

**git-rebase documentation overhaul complete** -- First-time contributor Julia Evans successfully landed her five-patch series restructuring the `git-rebase` man page to improve accessibility while preserving all technical content. The changes front-load practical examples, consolidate duplicate explanations, and better organize advanced topics like `--onto` usage. Maintainers helped refine terminology (standardizing on `git checkout` rather than `git switch` in examples) while praising the improved structure. The series serves as a model for documentation improvements that maintain technical precision while making complex features more approachable.

**What's cooking in git.git** -- Junio Hamano's regular status update covered 30+ topics across Git's development branches. Notable movements include several bugfixes graduating to 'master', continued progress on the `the_repository` removal effort (17 commits in ps/object-file-wo-the-repository), and new feature proposals like `git refs list` and `git last-modified`. The report highlights ongoing work in performance optimization, test improvements, and platform-specific fixes while flagging several topics needing rerolls or further discussion.

**Heap overflow fix confirmed** -- Gregoire Geis verified that Junio Hamano's solution for a `git diff --no-index` buffer overflow when comparing stdin in subdirectories works as intended. The fix properly handles path comparisons while maintaining expected behavior, with ASAN no longer reporting issues. This confirmation completes the validation cycle for a security-sensitive correction.

## In brief

**Test reorganization complete** -- D. Ben Knoble confirmed the help output testing reorganization has been approved for merging into `next`, concluding a multi-patch effort to refine Git's test strategy.

**git-jump filename handling** -- A follow-up confirmed the space-handling fix works for common cases while acknowledging an edge case with tab-terminated filenames remains.

**Cygwin test prerequisite** -- Junio Hamano squashed in a formatting tweak to make a Windows/Cygwin test skip message consistent with test-lib.sh conventions.

**for-each-ref documentation** -- René Scharfe standardized terminology for the `--start-after` option's argument across help text.

**merge --compact-summary fix** -- René Scharfe corrected an incorrect help string that misleadingly suggested the option took arguments.

## On the radar

**subtree split failure** -- A new bug report describes `git subtree split` failing on merge commits created by `git subtree add`, with reproduction steps provided but no technical analysis yet.