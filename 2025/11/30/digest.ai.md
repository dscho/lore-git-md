# Git Mailing List Digest - 30 November 2025

**The day in brief.** A moderately busy Sunday with 25 emails across 11 threads saw the conclusion of several significant discussions. Jeff King's ASan hardening series reached completion with new parsing utilities, Johannes Schindelin's Windows symlink compatibility work gained approval, and Derrick Stolee's Scalar documentation series neared integration. Performance optimizations and UI improvements also featured prominently.

## Notable threads

### **ASan hardening series concludes with new parsing utilities**  
Jeff King's 12-part series introducing safer integer parsing functions reached completion today with the final conversion of fsck.c to use the new `parse_unsigned_from_buf()` helper. The series establishes reusable parsing utilities that properly validate buffer bounds and numeric ranges, addressing gaps where previous ad-hoc implementations could trigger ASan complaints or miss overflow cases. The new functions in parse.c` now power cache-tree and fsck parsing with comprehensive unit test coverage. While the technical debate about code clarity versus minimalism may continue, Junio Hamano has approved the series for integration. In a related developer experience report, Jeff King documented several pain points with the clar unit test framework used for these tests, highlighting uninformative error messages and CI integration issues.

### **Windows symlink compatibility series approved**  
Johannes Schindelin's 10-part series preparing Git's test suite for Windows environments with proper symlink support received Junio Hamano's approval today. The final fixes addressed path handling in t7800-difftool.sh by matching Windows-style paths with trailing colons rather than assuming Unix forward-slash prefixes. The series as a whole tackles subtle platform differences in symlink behavior, permission handling, and path representation that would become visible when MSYS2 enables symlink support by default. Junio praised the solutions as "clever and clean" while acknowledging the broader challenge of maintaining cross-platform test compatibility when most developers work on Unix-like systems.

### **Scalar configuration documentation finalized**  
Derrick Stolee's series to comprehensively document Scalar's configuration settings is now complete pending minor editorial fixes. Today's exchanges confirmed resolution of Junio Hamano's review feedback, including typo corrections and clarification about `core.preloadIndex` behavior. The documentation spans 20+ settings across 239 lines of documentation, providing detailed rationales for performance tradeoffs and platform-specific behaviors. This concludes September's effort to properly maintain Scalar's configuration following its integration into core Git, having already addressed performance regressions and removed stale configs in earlier patches.

### **Performance optimization for diff-index**  
René Scharfe introduced a new `diff_same()` function that optimizes `git diff --find-copies-harder` by more efficiently handling unchanged file pairs. The change shows a measurable 1.33x speedup in Linux repository benchmarks by avoiding duplicate filespec allocations and redundant operations for unchanged files. Junio Hamano accepted the patch while noting the optimization might extend to `diff-tree` as well. This continues Scharfe's pattern of targeted, well-measured performance improvements to Git's core machinery.

## In brief

**`git clone` config parsing regression** -- The thread concludes with maintainers confirming the stricter whitespace handling in v2.52.0 will stand as an intentional bugfix, requiring affected tools like Atlassian Bamboo to adapt their command-line construction.

**`git last-modified` sparse checkout fix** -- Johannes Schindelin's fix for handling paths outside sparse checkouts prompts Derrick Stolee to question whether the command should validate paths more strictly even without the `--` separator.

**`git add -p` selection feedback** -- Abraham Samuel Adekunle's patch adding `[selected]` indicators to hunk prompts receives design feedback from Junio Hamano suggesting a more comprehensive solution across all interactive commands.

**`git repo structure -z` flag** -- A new patch adds null-terminated output support to `git-repo-structure` for consistency with other Git commands, completing the command's option parity with `git-repo-info`.

## On the radar

**`git diff --max-depth` merge commit bug** -- A new report identifies a fatal error when using depth limits with merge commits, where Git incorrectly attempts to diff tree objects as blobs. The edge case warrants investigation as it affects a useful debugging workflow.