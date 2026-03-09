# Git Mailing List Digest - 2026/01/04

**The day in brief.** A moderately busy Sunday with 44 emails across 19 threads, dominated by the conclusion of a long-running `git status` tracking branch feature and continued discussion of encoding issues in `.gitignore` files. Notable developments include the finalization of push tracking display in `git status` after 14 iterations, a regression fix for `git subtree`, and ongoing debate about Git's handling of non-UTF-8 encoded exclude files.

## Notable threads

### `git status` push tracking display finalized

After 14-iteration journey spanning multiple weeks, Harald Nordgren's feature to show push tracking information in `git status` output has reached completion. The final version cleanly displays divergence from both upstream and push tracking branches when they differ, using Git's existing `branch.<name>.pushRemote` configuration. The implementation includes comprehensive test coverage (210 lines) and handles all edge cases including custom refspec transformations. Junio C Hamano provided final review feedback on the preparatory refactoring patch, questioning some parameter passing but ultimately approving the technical direction. The series demonstrates Git's rigorous review process, with Phillip Wood's detailed technical review helping resolve refspec mapping issues and ensuring the change maintains Git's existing conventions.

### `git subtree` regression identified

A regression in `git subtree` functionality introduced in Git 2.52.0 was narrowed down to commit 83f9dad7d6 ("subtree: fix split after annotated tag was squashed merged"). The issue manifests when dealing with squashed subtree operations, where the new version fails to properly reconstruct history by only examining immediate commits rather than traversing ancestry. The reporter provided a minimal reproduction script showing how the change broke operations on repositories containing multiple subtrees where one was added via linear squash commit. Performance metrics show the regression also introduced a slowdown, with tree objects remaining correct but history reconstruction failing. The thread is now focused on determining whether to revert the problematic commit or modify it to restore the previous behavior while maintaining its original bugfix intent.

### Encoding handling in `.gitignore` files debated

A proposed patch to handle non-UTF-8 encoded `.gitignore` files (particularly UTF-16LE with BOM from Windows PowerShell) sparked significant discussion about Git's encoding assumptions. Junio C Hamano initially questioned the patch's fundamental assumption that Git universally expects UTF-8 encoded exclude files, noting behavior should align with platform-native encoding. Torsten Bögershausen later suggested the issue might be better addressed through user education rather than code changes, while brian m. carlson demonstrated Git's historical ability to handle arbitrary byte sequences in exclude files. The thread highlights tension between Windows usability (where UTF-16LE files are common) and Git's cross-platform design principles, with no clear resolution yet on whether to treat this as a Windows-specific edge case or pursue broader encoding policy changes.

### xdiff refactoring for Rust interop reviewed

Ezekiel Newren's 10-patch series refactoring xdiff internals to improve code organization and enable future optimizations (particularly around Rust interoperability) received maintainer feedback. Junio C Hamano noted general approval but raised concerns about placing Rust/C interop code in `compat/`, suggesting a more appropriate directory structure might be needed as the Rustification effort progresses. Subsequent review focused on coding style issues in the new `ivec` (interoperable vector) implementation, with Junio requesting use of Git's standard `FREE_AND_NULL` macro and `/* */` comment style rather than C++-style `//` comments. The technical style fixes are straightforward, leaving the architectural approach otherwise approved.

## In brief

**Reftable iterator fix** -- Tsahi Elkayam corrects undefined behavior in the indexed table reference iterator by properly validating union member access, addressing a known "BUG" marker in the code. Junio noted a formatting irregularity that needs correction before merging.

**Test modernization** -- Pushkar Singh's series converting shell `test` commands to helper functions in t1300-config.sh and t2021-checkout-overwrite.sh reached v4, now addressing all feedback about commit message structure and terminology.

**Difftool test fix** -- A racy test in t7800-difftool.sh was updated to avoid same-size file comparisons that were failing intermittently on Windows due to filesystem behavior, with Johannes Schindelin providing final polish on commit message conventions.

**Pack protocol documentation** -- Heath Dutton clarified the pack protocol specification by adding a missing definition for 'timestamp' (as '1*DIGIT') to match the existing 'depth' specification, with Junio confirming the technical accuracy.

**`git add -p` UI improvement** -- A GSoC student's patch to show previous hunk decisions in the interactive prompt was merged but required a follow-up fix for the PAGER test case that hadn't been updated to expect the new format.

**`git stash` data loss edge case** -- A bug report highlighted how `git stash`'s use of `reset --hard` can silently overwrite untracked directories that share names with previously tracked files, proposing `reset --merge` as a safer alternative.

## On the radar

**`git last-modified` performance** -- A bug report identified significant performance regression and directory handling issues in the relatively new `git last-modified` command, along with a crash when passed tree commit IDs directly.