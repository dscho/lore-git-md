# Git Mailing List Digest - 2025/09/03

**The day in brief.** A busy Wednesday with 90 emails across 25 threads, dominated by ongoing discussions about Rust adoption strategy, the new `git-history` command, and documentation improvements. Key developments include Junio Hamano weighing in on the gradual Rust transition plan and Julia Evans' `git-checkout` documentation series nearing completion after extensive review.

## Notable threads

### Rust adoption strategy debate continues

The long-running discussion about introducing Rust into Git saw significant movement today as Junio Hamano responded to Patrick Steinhardt's proposal for gradual adoption. Junio agreed with the cautious approach but raised practical concerns about which features could realistically be made optional without maintaining dual implementations. The thread also saw Ramsay Jones report concrete platform impact, having stopped building Git on Cygwin due to Rust's unavailability there. Collin Funk added historical context by comparing Rust adoption to Git's conservative approach to C99 features. The discussion remains focused on balancing technical benefits against platform support realities, with Junio's input helping shape a potentially viable transition path.

### `git-history` command refinement

Patrick Steinhardt's RFC series for the new `git-history` command saw active discussion today, particularly around the `split` subcommand's behavior. D. Ben Knoble reported issues with diff visibility during interactive splitting, while others suggested new subcommands inspired by vim-fugitive's squash/fixup workflow. The thread also included several implementation refinements: a helper function to reduce code duplication in the sequencer, documentation fixes for the `reword` subcommand, and cache-tree infrastructure improvements. While the core functionality appears solid, the discussion highlights ongoing work to polish the user experience and explore additional history editing operations.

### Reftable fsck validation review

Karthik Nayak's series adding fsck validation for the reftable backend received detailed review from Patrick Steinhardt and Shejialuo. The discussion focused on architectural questions about what validation belongs in fsck versus runtime code, with Junio Hamano suggesting some checks (like `tables.list` format validation) should be runtime requirements rather than fsck-only. Technical debates included error severity determination, string handling patterns, and test coverage improvements. The series is evolving from pure fsck checks toward potentially stricter runtime requirements, particularly for fundamental format violations.

### `git-checkout` documentation nears completion

Julia Evans' documentation series for `git-checkout.adoc` reached its final polishing stage after multiple rounds of review. Junio Hamano provided detailed line notes on phrasing, suggesting changes like replacing "left unchanged" with "changes follow you" for more intuitive descriptions. The v3 series restructures the DESCRIPTION section to be more accessible while maintaining all previously confirmed technical accuracy. With all major issues resolved, the discussion now focuses on fine-tuning pedagogical approaches to clearly convey checkout's transactional nature and file handling behaviors.

## In brief

**Color handling fixes** -- Jeff King's series addressing color configuration in interactive patch mode received review, with Patrick Steinhardt suggesting type safety improvements using `GIT_COLOR_UNKNOWN` instead of numeric values.

**Slab allocator cleanup** -- The slab allocator API series saw its final refinement, fixing a NULL check order issue in `alloc_state_free_and_null()` that could theoretically cause undefined behavior.

**Default branch test strategy** -- Phillip Wood and Patrick Steinhardt discussed test infrastructure for the default branch transition, agreeing to maintain `GIT_TEST_DEFAULT_INITIAL_BRANCH_NAME` while considering more branch-agnostic approaches.

**Upload-pack protocol fix** -- A two-part series addressed duplicate ACK behavior in protocol v2, first modernizing tests then fixing redundant acknowledgments for non-commit objects.

**Reference transaction hooks** -- Jeff King clarified that zero OIDs in hook output for ref updates are expected behavior when no old value is specified to the transaction, per githooks(5) documentation.

**Case-insensitive ref updates** -- Karthik Nayak's series fixing batched reference updates on case-insensitive filesystems received review from Junio Hamano, who suggested additional verification logic for case conflicts.

## On the radar

**`git whatchanged` deprecation** -- The discussion about transition paths continues, with Jeff King noting technical limitations preventing aliases from shadowing deprecated builtin commands during the deprecation period.

**Header file placement debate** -- Junio Hamano weighed in on the `git refs optimize` series, firmly stating that reusable components like `pack-refs.h` belong in the top-level directory rather than under `builtin/`.

**Midx-write error handling** -- Derrick Stolee's series fixing multi-pack-index write operations is nearly ready, with final discussions about error return patterns in `write_midx_internal()`.