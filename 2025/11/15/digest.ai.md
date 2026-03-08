Here's the Git mailing list digest for November 15, 2025:

## The day in brief

A moderately busy day with 45 emails across 12 threads, featuring significant progress on several technical fronts. The standout developments include final approvals for the mixed-hash submodule validation series and continued refinement of UTF-8 display alignment, while the xdiff refactoring discussion reached consensus on type documentation. A new bash prompt feature also entered the queue.

## Notable threads

**Memory leaks in receive-pack hook conversion**  
Junio Hamano identified two memory leaks in Adrian Ratiu's receive-pack hook API conversion - one in `feed_receive_hook_cb()` and a larger one in `find_receive_hook()` error paths. This marks the final polishing phase for the mature hook refactoring series, with attention now focused on edge case robustness before integration. The leaks were caught by CI testing, demonstrating the value of automated checks for this type of low-level plumbing work.

**xdiff type documentation consensus**  
After extensive discussion, the xdiff refactoring thread reached agreement on how to document C/Rust type mappings. Ramsay Jones, Ben Knoble, and Junio Hamano worked through precise characterizations of C's `char` type behavior (implementation-defined signedness versus "signless"), ultimately aligning on standard-compliant wording that maintains practical warning handling. Junio also suggested a small semantic correction for `get_indent()` to preserve character-specific operations despite the broader type safety changes.

**Submodule hash validation approved**  
brian m. carlson's v2 series preventing mixed SHA1/SHA256 submodules received final approval from both Jeff King and Junio Hamano. The solution adds validation check in `index_path()` with comprehensive test coverage, while Jeff's cleanup patch removes duplicate submodule resolution. This concludes a focused effort to enforce repository consistency, though broader SHA-256 transition questions remain open.

**UTF-8 alignment refinements**  
Jiang Xin's v2 series for CJK character alignment underwent detailed review, with Phillip Wood questioning encoding assumptions and Junio suggesting alternative implementation approaches using `printf()` padding. The discussion revealed systemic challenges with non-UTF-8 support across Git components. Jiang Xin addressed prior feedback by removing AI attribution trailers and adding focused test coverage, though the series now awaits decisions on the core alignment mechanism.

**Bash prompt quiet mode**  
Kiril Ivanov introduced a new 'quiet' option for Git's bash prompt upstream indicators, suppressing the '=' sign when branches are synchronized while maintaining divergence warnings. The well-structured three-part series modernizes tests and adds comprehensive coverage for both existing and new behaviors. Junio noted a tangential issue about bash shebang declarations in test scripts, but the feature itself appears ready for integration.

## In brief

**Lisp userdiff driver proposal** -- Scott L. Burson proposed a new "lisp" diff driver for Common Lisp, prompting discussion about whether to merge its patterns with the existing Scheme driver given their similarities.

**gitk tracking ref fix confirmed** -- Johannes Sixt verified that Git 2.52.0 fixes a regression where gitk crashed when displaying branches with deleted upstream references.

**`git whatchanged` replacement guidance** -- Kristoffer Haugsbakk clarified that `git log --no-merges --raw` fully replaces the deprecated `git whatchanged` command.

**`git fetch` tag handling follow-up** -- Karthik Nayak refined error handling for non-atomic fetches, proposing to run cleanup operations unless in strict atomic mode.

**`--committer` flag v4** -- ZheNing Hu's persistent proposal for symmetrical author/committer controls reached v4 with improved implementation but ongoing philosophical disagreement about its necessity.

## On the radar

**Experimental `git history` feedback** -- Early user reports on the new history rewriting interface are positive for basic "reword" operations, though architectural placement questions remain open.