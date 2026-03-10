# Git Mailing List Digest - 2026/01/25

**The day in brief.** A moderately busy Sunday with 27 emails across 11 threads, featuring documentation refinements, bugfix follow-ups, and lighthearted technical diversions. The most notable developments include final approval for the `git-last-modified` command's documentation, resolution of a regression in `git subtree` push operations, and continued progress on batched reference update error reporting.

## Notable threads

### `git-last-modified` documentation finalized

The recently merged `git-last-modified` feature has reached its final form, with only a minor documentation formatting issue identified by Kristoffer Haugsbakk. The command's behavior - now defaulting to non-recursive operation (max-depth=0) while maintaining backward compatibility via `-r/--recursive` - is fully documented and approved. While the pathspec markup nit will need a follow-up patch, the core functionality and documentation are complete after multiple iterations addressing input validation, error handling, and terminology.

### Histogram diff edge case fix under review

A bugfix for the histogram diff algorithm (XDF_HISTOGRAM_DIFF) in xdiff is being refined through review. The patch addresses an edge case where shifted change groups during compaction produce redundant diff output containing matching lines from both files. Phillip Wood and Junio Hamano are examining implementation details, including memory handling and whether a full re-diff is necessary versus just marking common prefixes/suffixes. The fix, which affects less than 1% of diffs, uses Myer's algorithm to correct suboptimal output in this specific scenario while preserving all original diff flags.

### `git subtree` push regression identified

A regression in `git subtree push` operations introduced in Git 2.53.0-rc1 has been confirmed by Junio Hamano. The issue, reported by Christian Heusel, causes valid no-op pushes to fail with "non-fast-forward" errors, breaking workflows like Arch Linux's AUR synchronization. Junio has committed to reverting the problematic commit (28a7e27cff) in 2.53.0-rc2 if a proper fix isn't available first. The prompt response reflects the impact on real-world usage patterns and the project's commitment to stability in release candidates.

## In brief

**Batched reference error reporting** -- Karthik Nayak and Jeff King's series restoring detailed error messages for batched reference updates concludes with a discussion about using `const char*` versus non-const pointers to indicate memory ownership, settling on the latter for consistency with project conventions.

**Documentation standardization** -- Jean-Noël Avila's man page conversion series receives detailed review feedback from Kristoffer Haugsbakk, identifying minor HTML rendering quirks in the `git-show` and `pretty-formats` documentation while confirming the technical correctness of the AsciiDoc conversion.

**String-list API improvements** -- Amisha Chhajed adds comprehensive test coverage for the string-list API in preparation for introducing `string_list_sort_u()`, a utility combining sort and deduplicate operations that will simplify several call sites.

**Git-gui translations** -- Alexander Shopov updates the Bulgarian translation for git-gui while Johannes Sixt ensures all `.po` files are treated as UTF-8 encoded, a minor but important consistency fix for the project's localization infrastructure.

**Missing objects proposal** -- Philip Oakley responds to Simon Richter's feature proposal for handling intentionally missing objects, referencing prior discussions and exploring technical challenges around maintaining repository integrity while allowing certain omissions.

## On the radar

**SHA-256 hash coincidences** -- The lighthearted thread exploring amusing patterns in SHA-256 outputs continues with Klaus Sembritzki optimizing Jeff King's brute-force search program, demonstrating SHA-256's cryptographic properties while maintaining the thread's playful tone.