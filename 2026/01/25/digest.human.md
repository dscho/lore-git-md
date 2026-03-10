# Git Mailing List Digest - 2026/01/25

**The day in brief.** A moderately busy Sunday with 27 emails across 11 threads, featuring documentation refinements, bugfix discussions, and a lighthearted exploration of SHA-256 hash properties. Notable items include final approval for the `git-last-modified` command's documentation and a confirmed regression in `git subtree` that will be fixed before the next release.

## Notable threads

### `git-last-modified` documentation finalized

The recently merged `git-last-modified` feature received its final documentation polish today. Kristoffer Haugsbakk identified a minor formatting issue in the man page where pathspec markup should use underscores rather than backticks in the synopsis section. While this correction will need a follow-up patch (since the feature is already in the `next` branch), it represents the last refinement for this command which changed its default behavior to be non-recursive (max-depth=0) while maintaining backward compatibility via `-r/--recursive`. The discussion also continued around the precise meaning of "revision" terminology in Git's error messages, with Patrick Steinhardt and Kristoffer Haugsbakk debating whether gitglossary(7)'s definition aligns with common usage.

### Histogram diff algorithm fix under review

Junio Hamano and Phillip Wood are reviewing a bugfix for the histogram diff algorithm (XDF_HISTOGRAM_DIFF) that addresses an edge case where shifted change groups produce redundant diff output. Phillip raised two key implementation questions: whether checking both file groups is necessary during compaction, and whether memory handling could be safer using direct assignment rather than sizeof-based memcpy. The patch already includes test coverage in t4073 with three cases verifying the fix, which uses Myer's algorithm to correct suboptimal output in this specific scenario. While this edge case affects less than 1% of diffs, the discussion shows careful attention to the patch's robustness and maintainability.

### `git subtree` regression confirmed

Junio Hamano confirmed a regression in `git subtree` introduced in Git 2.53.0-rc1 that causes valid no-op pushes to fail with "non-fast-forward" errors. This breaks real-world workflows, particularly Arch Linux package maintainers who use subtree pushes to synchronize between repositories. The problematic commit (28a7e27cff) modified subtree commit detection logic too aggressively. Junio stated the change will be reverted in 2.53.0-rc2 unless a proper fix emerges first, prioritizing stability for affected users. The regression was bisected and reported by Christian Heusel with clear reproduction steps showing how previously working pushes now fail.

## In brief

**Batched reference error messages** -- Karthik Nayak and Jeff King finalized their bugfix series restoring detailed error messages for batched reference updates, with a post-merge discussion about using `const char*` versus non-const pointers to indicate memory ownership.

**String list sort+deduplicate utility** -- Amisha Chhajed expanded test coverage for the string-list API in preparation for introducing `string_list_sort_u()`, a utility combining sorting and deduplication that will simplify several call sites.

**Documentation standardization** -- Jean-Noël Avila's man page conversion series received thorough review from Kristoffer Haugsbakk, noting successful standardization of `git-show` and `pretty-formats` documentation while identifying minor HTML rendering quirks.

**Git-gui translations** -- Alexander Shopov updated Bulgarian translations while Johannes Sixt ensured all `.po` files are treated as UTF-8 encoded, with Junio indicating these uncontroversial changes will be merged.

**SHA-256 hash coincidences** -- The lighthearted thread exploring amusing patterns in SHA-256 outputs continued with Klaus Sembritzki optimizing Jeff King's brute-force search program, demonstrating SHA-256's cryptographic properties remain robust despite playful optimization attempts.

## On the radar

**Missing objects proposal** -- Philip Oakley responded to Simon Richter's feature proposal for handling intentionally missing objects, referencing prior discussions from Git Merge 2020 while noting the technical challenges of maintaining repository integrity with omitted content. The discussion remains exploratory with no concrete solution yet.