# Git Mailing List Digest — 2025/03/30

**The day in brief.** A moderately active Sunday with 23 emails across 10 threads, featuring several long-running threads reaching their final polish stages. Key developments include the completion of Bash function detection improvements, resolution of reftable decoupling issues, and final refinements to `git blame` porcelain output. Documentation modernization continues with a new series converting command manuals to synopsis-style format.

## Notable threads

### Bash function detection improvements

Moumita Dhar's series to enhance Git's shell script function recognition in diffs reaches its final iteration (v4), addressing the last maintainer feedback about test file formatting. The changes improve detection of multiline definitions and line continuations while expanding coverage of shell syntax elements like parameter expansions and command options. After multiple rounds of technical implementation, this version focuses on ensuring proper newlines in test files — a small but important documentation detail that completes the review cycle. The patch appears ready for merging with comprehensive test coverage in t4018/ and t4034/.

### Reftable library decoupling finalized

The long-running effort to remove Git-specific dependencies from the reftable code concludes with Johannes Schindelin confirming that Patrick Steinhardt's workaround successfully addresses Windows-specific build issues. This clears the last blocker for the 18-commit series that systematically eliminates compatibility utilities (like BUG() and COPY_ARRAY()) from reftable code. Junio Hamano has marked the topic for promotion to 'next', representing a key architectural step in ref backend abstraction work.

### `git blame` porcelain output refinements

After extensive discussion about marking ignored/unblamable lines in `git blame` output, the thread reaches its final polishing stage. Phillip Wood identifies a potential output formatting issue where `puts()` calls could create redundant newlines, while also suggesting test improvements to use `test_cmp` rather than line counting. The core functionality — adding porcelain markers for lines affected by `--ignore-rev` — remains unchanged from earlier consensus, with these final touches addressing last-mile quality concerns before merging.

### Revision walker edge case fix

Matt Hunter's patch addressing `--left-only`/`--right-only` behavior with unrelated histories evolves through test improvements, with contributors refining the verification approach for better portability and robustness. The core fix (setting `revs->limited = 1` in revision.c) remains stable while the test case moves from Bash process substitution to temporary files and adopts Git's `test_line_count` helper. The discussion highlights interesting testing tradeoffs between precise output validation versus broader behavioral assertions.

## In brief

**Sign comparison warnings** in pathspec.c are cleaned up in a final version that removes the DISABLE_SIGN_COMPARE_WARNINGS macro while preserving meaningful negative values in array operations. **Git p4 encoding** receives a follow-up clarifying the handling of non-UTF-8 Perforce error messages, with Nikolay Shustov defending the class-based refactoring approach. A **security configuration question** surfaces about git-daemon ownership verification post-CVE, highlighting ongoing challenges with path resolution in public server setups. Alexander Shpilkin inquires about **stash functionality gaps** for build system integration, particularly the ability to snapshot untracked files without modifying the working tree.

## Documentation updates

Jean-Noël Avila's documentation modernization effort continues with a 5-patch series converting `git-reset`, `git-rm`, and `git-mv` manuals to use modern AsciiDoc synopsis blocks. The changes standardize placeholder notation and option formatting while fixing toolchain handling of special characters like backslashes and asterisks. These mechanical updates follow the established pattern for documentation improvements, with no behavioral impact.

## On the radar

The **Git Rev News** edition 121 draft is available for community review, celebrating the newsletter's 10th anniversary with planned publication on April 1. Contributors are invited to provide proofreading and content suggestions.