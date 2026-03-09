# Git Mailing List Digest - 2026/01/04

**The day in brief.** A moderately busy day with 44 emails across 19 threads, featuring the conclusion of a long-running `git status` enhancement, several test modernization patches, and discussions about encoding handling in `.gitignore` files. The standout development was the finalization of push tracking display in `git status` after 14 iterations of review.

## Notable threads

### Push tracking display in `git status` finalized

After 14 iterations of review, Harald Nordgren's series to show divergence from upstream and push tracking branches in `git status` output has reached its final form. The implementation now cleanly separates upstream and push tracking information, showing push divergence only when it differs from upstream. The feature uses Git's existing branch tracking configuration (`branch.<name>.pushRemote`) and includes comprehensive test coverage (210 lines) verifying various states and edge cases. Junio C Hamano provided final maintainer guidance on patch organization, emphasizing the importance of presenting a clean, logical progression in patch series. The thread also sparked a philosophical discussion about linear history versus merge-based workflows, with Junio clarifying that his preference is for coherent history rather than strictly linear history.

### Subtree regression with squashed merges

A regression in `git subtree` functionality when pushing subtree changes was narrowed down to commit 83f9dad7d6 ("subtree: fix split after annotated tag was squashed merged"). The reporter provided a minimal reproduction script demonstrating how the change broke subtree operations when dealing with linearized squash commits. The issue occurs when a repository contains multiple subtrees where one was added via a linear squash commit and another has changes committed after this linear squash commit. The new `should_ignore_subtree_split_commit()` logic fails to properly traverse ancestry when examining commit trailers. The reproduction case shows the split operation producing an incomplete history (fewer commits than expected) and confirms the issue doesn't exist in Git 2.43.0 (pre-dating the problematic commit).

### Encoding handling in `.gitignore` files

A proposed bugfix for handling non-UTF-8 encoded `.gitignore` files, particularly UTF-16LE files with BOM created by Windows PowerShell, generated significant discussion. Reviewers questioned whether Git should universally handle UTF-8 conversion for exclude files, suggesting this might be too broad a solution for what appears to be a Windows-specific edge case. brian m. carlson challenged the fundamental premise that Git assumes UTF-8 for exclude files, pointing out that Git historically allows arbitrary byte sequences to match Unix filename conventions. The discussion highlights a tension between Windows usability (where UTF-16LE files are common) and Git's cross-platform design principles, with no clear resolution yet on whether to treat this as a Windows-specific edge case or pursue a broader encoding policy change.

### Reftable iterator bugfix

First-time contributor Tsahi Elkayam submitted a bugfix for undefined behavior in the reftable backend's indexed table reference iterator. The patch properly validates `value_type` before accessing union members in `indexed_table_ref_iter_next()`, matching the safer pattern already used elsewhere in the codebase. While the technical content was quickly reviewed and approved by Pushkar Singh, Junio noted a formatting irregularity (two-space indentation instead of tabs in diff headers) that needs correction before merging. The thread shows a positive onboarding experience for a new contributor, with the project maintaining high standards for both technical correctness and formatting conventions.

### Xdiff refactoring for Rust interop

Junio C Hamano provided initial feedback on Ezekiel Newren's 10-patch series refactoring xdiff internals to improve code organization and enable future optimizations, particularly around Rust interoperability. Junio questioned the placement of the new `ivec` (interoperable vector) implementation files in `compat/`, suggesting that if more Rust-to-C interface files are expected, the project should consider establishing a more appropriate directory structure early on. In a separate review, Junio also noted coding style issues in the `ivec` implementation that need addressing to pass Git's automated style checks. The series remains under consideration with these organizational and stylistic points to resolve.

## In brief

**Test modernization** -- Pushkar Singh's series converting shell `test` commands to helper functions in t1300-config.sh and t2021-checkout-overwrite.sh reached v4, with only minor commit message formatting issues remaining before merging.

**`git add -p` UI improvement** -- A GSoC student's patch to show previous hunk decisions in `git add -p` was merged but required a follow-up fix for a test case that wasn't updated to expect the new prompt format.

**Pack protocol documentation** -- Heath Dutton clarified the pack protocol specification by adding a missing definition for 'timestamp' as '1*DIGIT', mirroring the existing format specification for 'depth'.

**`git stash` data loss edge case** -- A bug report highlighted how `git stash`'s use of `reset --hard` can silently overwrite untracked directories that share names with previously tracked files, proposing `reset --merge` as a safer alternative.

**Test suite cleanup** -- A small patch updated `t1420-lost-found.sh` to use `test_path_is_file` instead of raw `test -f` for better error reporting, continuing the test modernization effort.

## On the radar

**Documentation style consistency** -- Michael Lyons raised a question about whether to use backticks around option names in man pages, noting inconsistency between different Git commands that may warrant a standardization pass.

**`git last-modified` performance** -- A bug report identified performance and behavior issues in the `git last-modified` command, including significant slowdowns compared to equivalent `git log` commands and improper handling of directory paths.