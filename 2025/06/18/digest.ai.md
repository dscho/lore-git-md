# Git Mailing List Digest - 2025/06/18

## The day in brief

A moderately busy day with 43 emails across 13 threads, featuring significant progress on several fronts. The standout developments include a major performance optimization for `git fetch --prune`, resolution of platform compatibility issues in `diff --no-index`, and the finalization of a submodule remote lookup improvement series. Platform compatibility work dominated the day's technical discussions.

## Notable threads

### Major performance boost for `git fetch --prune`

Phil Hord's patch series addresses a severe O(N^2) performance problem in `git fetch --prune` when operating on repositories with many refs. The current implementation makes 2.6 billion strcmp calls when pruning 15,000 refs from a 174,000-ref repository, taking 470 seconds. The proposed change restructures the dangling ref detection to use an O(N*logN) approach, reducing runtime to under 1 second in test cases. The series has already garnered positive feedback from Jacob Keller, who explicitly supports the output ordering change (dangling ref warnings now grouped at end) as an improvement over the current interleaved output. The dramatic performance gains make this likely to be accepted despite the minor behavioral change.

### Platform compatibility fixes for `diff --no-index`

A regression in `diff --no-index` behavior on AIX and NonStop systems sparked a day-long discussion and iterative solution. The original issue stemmed from direct `d_type` access in `struct dirent`, which these platforms lack. Junio Hamano identified that the initial `DTYPE()` macro solution would incorrectly report all entries as non-directories. The final solution combines direct `d_type` checks where available with an `lstat()` fallback, ensuring correct directory detection for pathspec matching. Collin Funk verified the fix works on AIX 7.3, resolving both compilation and behavioral issues.

### Submodule remote lookup improvements finalized

Glen Choo and Jacob Keller's 7-part series to improve submodule remote lookup logic received maintainer approval in its third iteration. The series makes submodule remote handling more robust by: fixing memory leaks in branch/remote struct teardown, removing `the_repository` dependencies, and adding URL-based remote lookup as the primary method. The v3 simplification completely removes the `branch->merge_name` field, which Junio Hamano praised as a nice cleanup. The changes ensure submodule updates continue working even when users rename remotes, addressing a long-standing pain point.

## In brief

**Windows terminal handling refinement** -- James Duley and Carlo Arenas Belón continue discussing initialization semantics in Windows terminal handling code, debating whether zero-initializing `cmode_out` would improve reliability while addressing jurisdiction questions about main Git vs. Git-for-Windows fixes.

**Coccinelle compatibility improvements** -- Junio Hamano proposed changes to make Coccinelle rules more portable across versions, switching from regex pattern matching to explicit identifier lists in `commit.cocci`, with Collin Funk confirming the approach works well.

**Reftable block size configuration** -- Haylin Moore's investigation into network clone performance led to confirmation that `reftable.blockSize` is configurable, with Brian M. Carlson noting JGit uses 64KB blocks by default but cautioning about read performance tradeoffs.

**git-gui bug verification** -- Lauri Reilson confirmed Oswald Buddenhagen's March 2025 fix for commit message validation in git-gui works as intended, properly handling task-tracking syntax that was previously rejected.

**Test modernization** -- A v2 patch updates path checking in gettext tests to use Git's dedicated test helpers, addressing Patrick Steinhardt's feedback about better debugging output when tests fail.

**Documentation conversion complete** -- Junio Hamano approved Jean-Noël Avila's 9-part conversion of the `git-log` man page to the new synopsis format style, marking completion of converting Git's second-largest man page.