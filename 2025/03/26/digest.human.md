# Git Mailing List Digest — 2025/03/26

**The day in brief.** A moderately busy Wednesday with 63 emails across 20 threads, featuring significant progress on several fronts. The incremental MIDX bitmaps series nears completion, the comma operator refactoring reaches final stylistic debates, and Junio's "What's cooking" report highlights multiple topics graduating to `master`. The most notable development is the proposal of a new `git-blame-tree` command that could reshape how we analyze tree-level changes.

## Notable threads

### Release coordination between Git and Git for Windows

The long-running discussion about aligning release schedules between core Git and Git for Windows reached a potential resolution point. Johannes Schindelin expressed openness to alternatives after previous emails cited industry research against Friday releases, while Junio Hamano simplified the decision process with a straightforward "you say when, I'll agree" approach. The exchange revealed underlying coordination challenges but also both maintainers' willingness to find practical solutions. The technical resolution for the original Windows keyboard lockup bug (fixed in upcoming Git for Windows 2.49.0) remains unchanged.

### Incremental MIDX bitmaps ready for merge

Taylor Blau's incremental MIDX bitmaps series (tb/incremental-midx-part-2) received final approvals from both Elijah Newren and Jeff King after incorporating all v4 review feedback into v5. The 14-patch series implements performance optimizations for multi-pack indexes by adding bitmap layer tracking, EWAH iterator functionality, and pack-reuse support. With no new issues raised, this significant optimization appears ready for merging after multiple iterations.

### Comma operator refactoring reaches final debates

Johannes Schindelin's comprehensive series to eliminate questionable comma operator usage has reached its final stylistic debates after addressing all technical aspects. The discussion now centers on whether to use `(void)` casts (Johannes's preference) or explicit blocks (preferred by Phillip Wood and Taylor Blau) in `diff-delta.c` and `wildmatch.c`. Junio Hamano endorsed a `goto`-based approach for `wildmatch.c`, while the `diff-delta.c` question remains open. The series has already addressed build system integration (-Wcomma warnings) and most code changes across multiple subsystems.

### New git-blame-tree command proposed

Toon Claes introduced an ambitious 8-patch series proposing a new `git-blame-tree` command that traces file modifications through tree objects, analogous to line-level `git-blame` but operating at the tree level. Building on Jeff King's 2011 work, the series includes performance optimizations like pathspec tries and dynamic pathspec narrowing during traversal. Taylor Blau suggested an alternative Bloom filter-based approach from his work with Derrick Stolee, potentially sparking an architectural discussion about the best way forward for this long-desired functionality.

### What's cooking report highlights progress

Junio C Hamano's "What's cooking" report showed several notable topics graduating to `master`, including fixes for merge crashes involving renames, documentation markup improvements, and enhanced ref consistency checks in `git fsck`. New topics introduced include configurable batch size for loose object maintenance and various build system improvements. The report also highlighted ongoing efforts like path-walk delta compression and continued removal of `the_repository` dependencies, providing a comprehensive snapshot of Git's current state.

## In brief

Jeff King confirmed Taylor Blau's renaming of `bitmap_non_extended_bits` to `bitmap_num_objects_total()` in the MIDX bitmaps series, resolving earlier review feedback about unclear naming. The reftable library decoupling effort cleared its last Windows-specific build issue, with Johannes Schindelin confirming successful CI runs after Patrick Steinhardt's mimalloc relocation fix. Zheng Yuting's GSoC project on SMTP authentication error handling reached v9 with all technical aspects approved, now awaiting test suite resolution. Jayatheerth K's GSOC documentation series modernizing the MyFirstContribution tutorial is complete pending final pedagogical decisions about header inclusion explanations. Phillip Wood provided constructive feedback on Nikolay Shustov's git-p4 encoding fix, suggesting separating refactoring from bugfixes for easier review. Karthik Nayak's `git blame` porcelain output bugfix received test improvement suggestions from Eric Sunshine to make output more informative. A platform-specific test failure was reported for Derrick Stolee's `backfill --min-batch-size` feature on s390x and sparc64 architectures.

## On the radar

The interactive rebase parsing issue involving commit messages starting with "Drop" has shifted from being seen as a command parsing problem to a state management issue during operation changes, with Phillip Wood now gathering concrete reproduction steps. Ayush Chandekar submitted a promising GSoC proposal for refactoring Git's global state as part of the ongoing `the_repository` removal effort, demonstrating research into prior work and proposing concrete structural changes. The reftable lockfile handling discussion on Windows continues, with Johannes Schindelin raising concerns about using the undocumented `RtlGetLastNtStatus()` function and proposing alternative approaches to detect pending file deletions.