# Git Mailing List Digest - 2026/05/08 (Friday)

**The day in brief.** A moderately busy Friday with 54 emails across 18 threads, featuring significant progress on Windows platform improvements, documentation refinements, and several performance optimizations. Key highlights include Johannes Schindelin's completion of the nedmalloc removal series and large-object handling patches for Windows, plus substantive discussions about merge commit handling in `git history` and promisor remote configuration security.

## Notable threads

### Windows platform improvements finalized

Johannes Schindelin completed two major Windows-focused patch series today. The **nedmalloc removal** (v3) fully excises the unmaintained allocator that was causing GCC 16 compatibility issues, with all build system references and implementation files now deleted. The series received explicit approval from Patrick Steinhardt and Junio Hamano suggested integration approaches for the large file deletions.

Simultaneously, Schindelin's **large object handling** series (v3) addresses Windows' 32-bit type truncation issues when processing objects >4GB during cloning. The latest iteration incorporates type safety refinements from Torsten Bögershausen's review, including proper memory alignment handling. The series now includes comprehensive CI strategy (patch 11/11) to run expensive >4GB tests only on integration branch pushes.

### Merge commit support in `git history`

Phillip Wood provided substantive review of Johannes Schindelin's RFC series adding merge commit support to `git history`. The discussion focused on edge cases in conflict resolution preservation, particularly when rewritten parents introduce new conflicts not present in the original merge. Wood proposed an alternative approach for single-parent rebase cases that may simplify the implementation while maintaining correctness. This technical dialogue continues to shape the feature's design as it moves toward finalization.

### Promisor remote security mechanism

Christian Couder's URL-based promisor remote configuration series received detailed review from Toon Claes on its security mechanism (patch 6/8). The review examined edge cases in URL pattern matching, user/password field handling, and test consistency. The feedback demonstrates the careful attention being given to security implications as this feature nears completion, with all prior approvals from Patrick Steinhardt and Junio Hamano still standing.

## In brief

**git-interpret-trailers documentation** -- Kristoffer Haugsbakk's 9-patch series improving trailer block parsing documentation is now complete, having addressed all review feedback. The changes clarify strict key format requirements and standardize terminology.

**rebase --update-refs edge case** -- A v2 bugfix addresses incorrect update-ref instructions for non-branch refs when rebase.instructionFormat shows decorations. Phillip Wood approved the refined test case.

**git-jump automatic mode selection** -- Greg Hurrell's contrib script improvement now automatically selects merge or diff mode based on repository state. After substantive UX discussion with Jeff King, the patch reached consensus and is ready for merging.

**HTTP fetch test pollution** -- Junio C Hamano fixed test interference in t5551 when GIT_TEST_LONG is enabled, with Jeff King confirming the cleanup approach as "obviously correct".

**merge-base performance optimization** -- A new patch optimizes single merge-base calculation in large repositories using generation numbers, showing 100x speedups in some cases while maintaining correctness.

**git format-rev NUL-handling** -- Kristoffer Haugsbakk refined documentation for the new command's text-mode NUL handling, focusing explanations on the most relevant use cases.

**per-worktree ignore patterns** -- D. Ben Knoble submitted a documentation-only patch clarifying that per-repository ignore patterns use $GIT_COMMON_DIR, scaling back from an earlier implementation attempt after maintainer feedback.

## On the radar

**Maintenance subsystem resource usage** -- Jeff King's analysis of unexpected resource consumption during `git add`/`commit` identified maintenance operations running independently of gc.auto settings. The thread awaits input from Derrick Stolee on potential locking mechanisms.

**--track=fetch refinement** -- Harald Nordgren's series extending --track with fetch mode received detailed technical feedback from Phillip Wood about edge case handling, moving the discussion beyond earlier philosophical objections.