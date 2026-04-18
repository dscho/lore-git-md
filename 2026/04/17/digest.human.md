# Here's the daily digest for April 17, 2026:

**The day in brief.** A moderately busy Friday with 45 emails across 17 threads, featuring final preparations for Git 2.54.0-rc, significant test infrastructure improvements, and ongoing discussions about source tree structure and worktree submodules. The highlight is Patrick Steinhardt's completed series enabling `set -e` in the test suite, while Junio's "What's cooking" report provides a comprehensive snapshot of current development efforts.

## Notable threads

### Test suite modernization with `set -e`

Patrick Steinhardt's 12-patch series to enable strict error checking (`set -e`) in Git's test suite for Bash 5+ is now complete. The changes systematically prepare the test framework by adapting helper functions and fixing test patterns that would break under strict error checking. Key adaptations include making `git config --unset` resilient to missing keys, converting test patterns to proper conditionals, and updating test helpers like `test_must_fail()` and `test_when_finished()`. The series has undergone thorough review from Jeff King, SZEDER Gábor, and Junio Hamano, with CI validation confirming Bash 5+ compatibility. This represents a significant improvement in test robustness by catching errors that would previously be silently ignored.

### Source tree reorganization RFC

Patrick Steinhardt's RFC proposal to move libgit.a components into a dedicated "lib/" directory gains support from key contributors. Elijah Newren generally approves the reorganization but notes a minor concern about moving the sha1collisiondetection submodule. brian m. carlson clarifies Rust build requirements, noting `build.rs` must remain at the root while source files could move to a "rust/" subdirectory. Junio C Hamano supports the change, citing improved tab completion as a concrete benefit. The discussion shows consensus forming around this significant structural change, with only minor implementation details remaining to be resolved.

### Worktree submodule object sharing

Phillip Wood identifies a critical flaw in his proposed `commondir`-based approach for sharing submodule objects between worktrees - `git gc` would miss per-worktree refs and index files, potentially causing premature object pruning. This strengthens Junio Hamano's earlier suggestion of using a single bare submodule repository in `$GIT_DIR/modules/`. The technical discussion continues to evaluate tradeoffs between implementation complexity, platform compatibility, and proper GC behavior, with Junio's approach now appearing more robust.

## In brief

**UTF-8 diffstat truncation fixes** -- Elijah Newren provides v2 of his patch fixing NULL dereference and out-of-bounds read vulnerabilities in diffstat filename display, introducing a new utf8_ish_width() wrapper that safely handles invalid UTF-8 and control characters.

**Bisect terminology consistency** -- A patch series (v3) ensures custom bisect terms like "old/new" are consistently used in status messages and `git rev-parse --bisect` output, with Junio Hamano raising thoughtful questions about localization implications.

**Promisor repack final polish** -- Lorenzo Pegorari addresses final review feedback on his promisor file handling series, hardening the `copy_promisor_content()` helper with better input validation and error handling.

**MIDX version handling** -- Junio and Jeff King confirm alignment on MIDX error handling approach for Git 2.54.0, deferring deeper improvements to post-release development.

**Git 2.54.0-rc updates** -- Final pre-release fixes include documentation improvements, MIDX version revert, and CI updates, with Elijah Newren reporting late-found edge cases that may warrant 2.54.1 follow-up.

## On the radar

**Rust build system integration** -- brian m. carlson's clarification about Rust's `build.rs` requirements may influence the final source tree reorganization plan, ensuring both shell usability and cargo compatibility.

**Subtree split improvements** -- Colin Stagner's analysis of merge handling in `git subtree split` continues to explore ways to make the command more deterministic while preserving history.