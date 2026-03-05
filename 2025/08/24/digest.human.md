# Git Mailing List Digest - 2025/08/24

**The day in brief.** A busy Sunday with 51 emails across 13 threads, dominated by continued refinement of Patrick Steinhardt's ambitious `git-history` RFC series (now at v3) and performance optimizations for `git describe` and line-log. Notable developments include consensus forming around the signal-free progress meter approach and a bugfix for fetch-pack race conditions.

## Notable threads

### `git-history` command RFC evolves to v3

Patrick Steinhardt's proposal for a new `git-history` command family (inspired by Jujutsu's history editing model) saw extensive discussion as it approaches finalization. The series now includes four subcommands (`drop`, `reorder`, `split`, and new `reword`) with management commands for interrupted operations. Key developments today:

- Confirmed `post-rewrite` hook integration will be added in v3, addressing Kristoffer Haugsbakk's concern about consistency with other rewrite operations
- Refined documentation based on Jean-Noël Avila's feedback, particularly around parameter naming (`<commit>` vs `<revision>`) and synopsis formatting
- Clarified behavior comparisons with Jujutsu's equivalent commands, confirming automatic descendant rebasing will be included
- Junio C Hamano advocated for showing original commit messages in both resulting commits during splits to preserve context
- Kristoffer Haugsbakk questioned the use-case for imperative `reorder` versus interactive rebase, prompting discussion about lightweight vs. full-featured workflows

The series has progressed through careful infrastructure preparation and now shows signs of stabilizing, with most discussion focused on UX polish rather than fundamental design questions.

### Performance optimizations for `git describe`

René Scharfe and Jeff King debated competing implementations to optimize `git describe`'s depth computation, which currently has quadratic complexity. Scharfe's khash-based approach showed a consistent 3% speed advantage over King's commit-slab alternative (both ~1.3x faster than original), prompting discussion about broader architectural tradeoffs:

- Khash uses less memory for sparse cases but requires key storage
- Commit-slabs offer zero-cost lookup but allocate for all commits
- Both approaches significantly outperform the current linear scan
- Discussion extended to potential elimination of the `commit->index` field through pointer hashing

The exchange highlighted how even small performance differences can surface important questions about Git's core data structure strategies.

### Line-log optimization series

An unnamed contributor submitted a 4-part series optimizing `git log -L` merge handling, with:
1. A 2.25x speedup for merge-heavy histories by avoiding unnecessary tree diffs
2. Three follow-up cleanups simplifying memory management and initialization
3. Benchmarks showing significant improvements in both git.git (25.8% merges) and linux.git (7.5% merges)

The changes are purely internal optimizations with no user-facing impact, continuing Git's tradition of incremental performance improvements to core commands.

## In brief

**Fetch-pack race condition fix** -- Resolves an edge case where fetches could incorrectly conclude objects were missing if they were recently repacked, by forcing packfile rescanning with `HAS_OBJECT_RECHECK_PACKED`.

**Progress meter delay bugfix** -- Johannes Sixt corrected incorrect flag handling that caused `GIT_PROGRESS_DELAY` to always use 1 second, while also updating the default to match this de facto behavior.

**Subtree split regression fix** -- Addresses a bug where squashed subtree merges under a prefix were incorrectly ignored during splits, with comprehensive test coverage added.

**`git diff --index` proposal** -- Suggests adding `--index` as an alias for `--cached` to standardize terminology, though deprecation of `--cached` would be a separate step.

**Documentation typo fix** -- Kristoffer Haugsbakk corrected an errant backtick in mergetool configuration docs, maintaining consistency with Git's documentation style.

## On the radar

**Meson build system integration** -- The discussion about when to declare Meson officially supported continues, with Patrick Steinhardt and Ramsay Jones debating whether remaining issues (like quoting problems) warrant keeping experimental status.

**Signal handling modernization** -- Johannes Sixt's signal-free progress meter approach appears to be gaining consensus, with Junio Hamano now suggesting optimizations to minimize `getnanotime()` overhead.