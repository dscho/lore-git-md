# Git Mailing List Digest - 2026/01/06

**The day in brief.** A busy Tuesday with 79 emails across 27 threads, featuring significant progress on multiple fronts. Key developments include final reviews for the submodule gitdir path encoding series, architectural refinements to `git-history` command, and performance optimizations for `git cat-file`. Several bugfix series reached merge-ready status while documentation and test infrastructure improvements continued apace.

## Notable threads

### Submodule gitdir path encoding reaches final polish

Patrick Steinhardt's long-running series to make submodule gitdir paths configurable at runtime has reached its seventh iteration with all major technical concerns resolved. The implementation introduces a new `extensions.submodulePathConfig` extension that enables runtime configuration via `submodule.<name>.gitdir`, replacing build-time configuration. Reviewers confirmed the architectural soundness, with only minor documentation and test formatting nits remaining. The series now awaits final merging after addressing these last polish items.

### git-history command scope refined

The proposed `git-history` command's branch rewriting behavior saw significant refinement through discussion between Patrick Steinhardt, Elijah Newren, and Phillip Wood. Consensus emerged to limit rewrites to local branches only (`refs/heads/*`), excluding remote-tracking refs and tags to respect Git's immutability boundaries. Performance concerns about branch walking were also addressed by adopting Wood's analysis showing repositories typically have few local branches compared to other ref types. These changes narrow the feature's scope while making it more predictable and performant.

### Memory management fixes ready for merging

Two independent memory management fixes from Patrick Steinhardt received final review acks:
1. A commit-graph writing fix in `builtin/gc.c` that corrects memory leaks and object counting issues
2. A submodule ODB leak fix in `odb.c` ensuring proper cleanup sequence

Both patches have gone through multiple iterations with thorough review from Justin Tobler, Karthik Nayak, and Toon Claes. The series is now considered technically sound and ready for inclusion, with all feedback addressed and comprehensive test coverage in place.

### Performance optimization for cat-file bitmaps

Jeff King identified and fixed a performance regression in `git cat-file` where bitmap indexes were unnecessarily loaded even for unfiltered operations, causing a 331% slowdown in some cases. The fix modifies `builtin/cat-file.c` to only prepare bitmaps when an object filter is active, restoring original performance for unfiltered operations while maintaining the optimization benefits for filtered cases. Clear benchmark evidence demonstrates both the problem and solution.

## In brief

**Reftable compaction fix** -- Patrick Steinhardt corrects a compaction edge case that could silently drop refs when two tables share a deletion tombstone.

**French translation update** -- Jean-Noël Avila brings the French `.po` file up to date with the latest source strings.

**Interactive add-patch UI improvement** -- A GSoC project adds visual feedback for previous hunk decisions in `git add -p`, now in its sixth iteration with prompt formatting refinements.

**HTTP auth test coverage** -- Ashlesh Gawande adds test cases for netrc credential handling with 401/403 responses, addressing Junio's style feedback in v2.

**Performance test fixes** -- Jeff King corrects output directory assignment and environment variable preservation in Git's performance test infrastructure.

**git diff --relative bug** -- Johannes Sixt reports unexpected behavior when `--relative` treats its argument as a text prefix rather than directory path.

## On the radar

**Bazel build support** -- Discussion continues about adding Bazel support, with the author now planning an out-of-tree approach via Bazel Central Registry due to maintainer concerns about in-tree maintenance burden.

**gitattributes config timing** -- The `the_repository` removal effort surfaces challenges around lazy vs eager config validation, particularly for `core.attributesFile` during interactive rebase.

**Exclude file encoding** -- Debate continues about whether `.gitignore` files should handle UTF-16LE BOMs from Windows or maintain byte-sequence matching behavior.