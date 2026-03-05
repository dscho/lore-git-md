# Git Mailing List Digest - 2025/08/24

**The day in brief.** A busy Sunday with 51 emails across 13 threads, dominated by continued refinement of Patrick Steinhardt's `git-history` RFC series and several performance optimizations. Key developments include progress on the new history editing commands, a fix for fetch-pack race conditions, and line-log optimizations for merge commits.

## Notable threads

### `git-history` command RFC v3

Patrick Steinhardt's RFC series introducing Jujutsu-inspired history editing commands saw extensive discussion today, with the series now at v3. The proposal adds four subcommands (`drop`, `reorder`, `split`, `reword`) and management commands (`abort`, `continue`, `quit`), building on Git's sequencer infrastructure. Key developments:

- Confirmed `post-rewrite` hook integration will be added in v3
- Refined documentation based on Jean-Noël Avila's feedback
- Addressed Kristoffer Haugsbakk's questions about `reorder` vs interactive rebase use cases
- Fixed an error message that incorrectly referenced "split" in the `reword` context
- Added infrastructure for in-memory index operations needed by `split`

The series appears to be stabilizing, with Junio having approved the conservative implementation approach. Remaining discussion focuses on edge cases and usability refinements rather than fundamental design questions.

### Fetch-pack race condition fix

A bugfix addresses a race condition in fetch-pack where objects existing in the commit-graph but recently repacked could be incorrectly reported as missing. The patch adds `HAS_OBJECT_RECHECK_PACKED` to force rescanning packfiles when needed, fixing cases where concurrent repacks could cause fetches to fail unnecessarily. While discovered in partial clone scenarios, the fix benefits all fetch operations running alongside repacks.

### Line-log optimization series

A four-part series optimizes `git log -L` performance for merge-heavy histories by eliminating unnecessary tree diffs during merge commit processing. The main optimization (patch 1) shows 1.3-2.25x speedups by combining diff computation and processing into a single pass. Subsequent patches clean up the implementation by removing unnecessary allocations, fixing initialization safety, and simplifying condition checks.

## In brief

**Progress meter delay fix** -- Johannes Sixt corrects a long-standing bug where `GIT_PROGRESS_DELAY` was ignored, always waiting just 1 second. The patch also updates documentation to match the de facto behavior.

**`git describe` performance** -- René Scharfe and Jeff King discuss alternative optimizations (khash vs commit-slabs) for depth computation, with both approaches showing ~1.3x speedups over the original implementation.

**`git diff --index` proposal** -- A new flag is proposed as an alias for `--cached`, aiming to standardize terminology with other commands like `git add` and `git rm`.

**Subtree split fix** -- A regression in handling squashed subtree merges under prefixes is fixed, ensuring nested subtrees are properly processed during splits.

**Documentation typo** -- Kristoffer Haugsbakk corrects an errant backtick in mergetool configuration docs, restoring proper apostrophe usage.

## On the radar

**Rust FFI design** -- Ben Knoble's questions about `ivec` implementation choices in the xdiff Rustification effort remain open, probing the balance between Rust and C code.

**Meson build status** -- The discussion about when to declare Meson as officially supported continues, with Ramsay Jones cautioning about outstanding issues while Patrick Steinhardt maintains core functionality is complete.