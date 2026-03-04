# Git Mailing List Digest - 2025/08/05

**The day in brief.** A busy day with 122 emails across 24 threads, featuring several major developments. The highlight is the completion and merging of the `git last-modified` command series after extensive review, while significant progress was made on reflog migration fixes, merge-ort rename detection, and documentation standardization. The Git for Windows 2.51.0-rc0 release candidate was also announced.

## Notable threads

### `git last-modified` command merged

After seven iterations and extensive review, Toon Claes' `git last-modified` command series was finalized and merged today. This new command shows when files were last modified in a repository's history by walking backwards to find commits where paths transitioned to their final state. The implementation includes Bloom filter optimizations (50% faster for top-level queries), comprehensive test coverage, and careful handling of edge cases around commit graph handling and memory management. The series also contributed to clarifying `_release()` vs `_clear()` conventions in Git's coding guidelines. Documentation received final polish with backtick formatting standardization.

### Reflog migration infrastructure completed

Patrick Steinhardt's 9-patch series addressing reflog migration issues between files and reftable backends reached its final form in v5. The changes correct committer identity handling, fix an all-zero old OID issue affecting libgit2 compatibility, add a new `git reflog write` subcommand, and implement comprehensive race condition handling for HEAD updates. After extensive discussion about whether to silently skip or explicitly abort on detected HEAD races, the maintainers converged on the simpler abort approach as being more maintainable despite the rarity of these race conditions. The series is now ready for merging into Git 2.51.0.

### Merge-ort rename detection fixes

Elijah Newren submitted v2 of his 6-patch series fixing edge cases in merge-ort's rename detection, particularly involving directory renames that affect unrelated files. The changes address three distinct bugs that occur when files are renamed to themselves (A->A) during merges: silent deletions when files should be kept, incorrect retentions when files should be deleted, and wrong index entry counts. The series includes extensive new test coverage in t6423-merge-rename-directories.sh (+820 lines) and resolves real-world merge failures observed at GitHub. After resolving some test assertion philosophy differences with Patrick Steinhardt, the series appears ready for merging.

### Documentation standardization

A comprehensive 6-patch series introducing automated linting tools for Git's man pages was merged. The changes add Perl scripts to check for:
1. Proper `linkgit:` macro usage
2. Correct AsciiDoc section delimiters
3. Single-term definition list entries
4. Split `--[no-]parameter` forms
5. Standardized synopsis formatting
6. Backtick-wrapped option names

The linters enforce consistency across 57 documentation files while maintaining compatibility with Jean-Noël Avila's parallel synopsis conversion work. This systematic approach to documentation quality control through automated checks received positive reviews and represents a significant step forward in maintaining documentation standards.

## In brief

**Git for Windows 2.51.0-rc0** -- Johannes Schindelin announced the release candidate based on Git v2.51.0-rc0, with updates to 7-Zip 25.00, cURL 8.15.0, and MSYS2 runtime.

**Translation updates for 2.51.0** -- Jiang Xin kicked off the localization effort with 58 new messages needing translation, setting an August 16 deadline for contributions.

**`git refs list` subcommand approved** -- Meet Soni's GSoC project to create a `git refs list` wrapper around `for-each-ref` was approved after addressing all review feedback and rebasing onto 2.51.0-rc0.

**`git sparse-checkout clean` edge cases** -- Elijah Newren identified several documentation ambiguities around which files get cleaned, particularly regarding staged vs unstaged changes, prompting further test refinement.

**`git mv --after` proposal closed** -- After extensive discussion, the thread concluded that Git's content-tracking design fundamentally differs from operation-tracking systems, making explicit move tracking incompatible with Git's model.

**On the radar**

**`git cat-file --batch-command` performance** -- Rob Browning reported severe performance issues (32 hashes/sec) in large repositories, though follow-up tests suggest this may be hardware-specific to external SSDs rather than a Git regression.