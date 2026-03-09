# Git Mailing List Digest - 2025/06/09 -- 2025/06/15

## The week in brief

A busy week with 325 emails across 108 threads saw significant progress on multiple fronts. The period was dominated by the finalization of major features like `imap-send` overhaul and stash import/export, while platform compatibility discussions and `the_repository` removal efforts continued steadily. Key developments included Git v2.50.0-rc2's release with notable performance improvements, ongoing debates about tag creation behavior and memory management philosophy, and several bugfixes reaching completion.

## Key developments

### `imap-send` resurrection complete

After 18 iterations, Aditya Garg's comprehensive `imap-send` series is ready for merging. The final version fixes a critical configuration parsing regression (broken since Git 2.46.0), adds OAuth2.0 authentication support (both RFC-standard OAUTHBEARER and Google's XOAUTH2), implements proper PLAIN authentication for OpenSSL, and introduces new folder management features (`--folder` override and `--list` enumeration). The series also addresses numerous memory leaks and improves user experience with better credential prompts. Junio Hamano approved the core changes after minor documentation and style nits were addressed in the final version.

### Tag creation behavior debate

A lively discussion emerged around improving tag conflict handling, with three competing proposals: showing more detailed error messages when tags exist (original suggestion), keeping the current simple "tag exists" message (Junio's preference), and making tag creation idempotent when commits match (Hilco Wijbenga's proposal). The thread clarified technical constraints around preserving tag metadata in idempotent operations and established important considerations about tag type differences (lightweight vs annotated). While no consensus was reached, Junio later proposed a technical implementation making lightweight tag creation idempotent when pointing to the same commit, while maintaining existing behavior for annotated tags.

### Git v2.50.0-rc2 released

Junio announced the second release candidate for Git 2.50, featuring 604 non-merge commits from 85 contributors. Highlights include new `--combine-cruft-below-size` for `git repack`, TCP keepalive configuration for HTTP, `git reflog drop` subcommand, machine-parsable `rev-list` output, and performance improvements in ref handling and reftable operations. The release continues Git's modernization efforts with MIDX bitmaps, reftable optimizations, and reduced `the_repository` usage, alongside numerous Windows/ARM64 fixes and test improvements.

### Stash import/export feature finalized

brian m. carlson submitted the eighth and final version of the stash import/export series, addressing the last two review items from Phillip Wood. The implementation now properly handles ownership semantics in `write_commit_with_parents()` and invalid argument combinations for `export`. The series introduces new `git stash export` and `import` subcommands that enable bidirectional transfer of stashes between repositories, storing commit chains under `refs/stash-export/`. With 268 lines of test coverage and positive reviews from multiple contributors, this long-running series appears ready for merging.

### Memory management philosophy debate

What began as a straightforward memory leak fix in the notes subsystem evolved into a broader discussion about Git's error handling philosophy. Jeff King (peff) strongly challenged Junio Hamano's position that memory should be explicitly freed before `die()` calls to keep sanitizer output clean. Peff argued that modern tools properly categorize such cases as "still reachable" rather than leaks, and that attempting to free all allocations before `die()` would be impractical. By week's end, Junio endorsed Jeff's position, establishing clearer guidelines for future error path implementations.

## In brief

**gitk external diff rename handling** -- Tobias Boesch's five-iteration effort to fix gitk's external diff functionality for renamed files reached completion, replacing regex-based filename parsing with direct `git diff --find-renames` execution.

**Promisor-remote protocol validation** -- Christian Couder's 5-patch series extending the promisor-remote protocol to support configurable validation of remote attributes reached its final form (v4) after extensive review.

**Batch reference updates** -- Patrick Steinhardt's bugfix series for batched reference updates completed its journey through review, fixing a files backend segfault when handling failed updates.

**Solaris build fixes** -- Multiple contributors diagnosed and fixed build issues on Solaris where older sed implementations don't support the -E flag for extended regex.

**Compact merge summaries** -- Junio introduced a new `--compact-summary` option for merge/pull operations that modifies diffstat output to explicitly mark created/deleted paths.

**`git repo-info` proposal** -- Lucas Seiki Oshiro introduced an RFC for a new `git repo-info` command as part of a GSoC project, aiming to split repository metadata functionality from `git rev-parse`.

**FreeBSD version support** -- Discussion established FreeBSD 6.0 as the new minimum supported version, removing several legacy build settings that were only needed for FreeBSD 4.x systems.

**Looking ahead**

Several topics are poised to dominate next week's discussions:

- The ongoing debate about implementing batch operations in `git branch` vs `update-ref` continues, with Junio clarifying his objection to coupling the feature with `--force` behavior
- The RFC for `git refs list` represents a significant change to how users interact with Git's reference system, though its future remains uncertain after Junio's skepticism
- Error handling patterns may see broader discussion following the resolution of the memory management debate
- Platform compatibility work will likely continue with Solaris CI testing discussions and FreeBSD version enforcement