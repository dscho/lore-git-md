# Git Mailing List Digest - 2025/05/02

**The day in brief.** A moderately active Friday with 65 emails across 22 threads, featuring significant progress on several fronts. The path-walk delta compression series nears final approval, the long-running `git gc` decomposition effort reaches completion, and multiple test modernization efforts advance. Junio Hamano's "What's cooking" report highlights several topics ready for merging.

## Notable threads

**Path-walk delta compression approved for 'next'** -- Junio Hamano marked Derrick Stolee's path-walk delta compression series as ready for inclusion in 'next' after it received no objections during its review period. The feature introduces path-based delta compression grouping through `--path-walk` and `pack.usePathWalk` config, with optimizations for shallow clones. Taylor Blau is conducting a final review pass, providing detailed feedback on documentation, test robustness, and implementation details like thread safety and config handling. Performance data shows dramatic improvements (100x size reduction in some cases) for repositories with path-based similarity patterns.

**`git gc` decomposition completed** -- Patrick Steinhardt's series to fully decompose `git gc` into standalone maintenance tasks reached its conclusion with the addition of worktree pruning and rerere garbage collection. The v3 iteration simplified the rerere GC implementation in response to Junio Hamano's performance concerns, replacing a three-phase staleness check with a simpler directory entry count. Derrick Stolee confirmed the series is ready for merging, marking the end of a multi-year effort to replace the monolithic `git gc` with granular, configurable maintenance operations.

**Deprecation infrastructure established** -- A new standardized deprecation process is being introduced, with `git whatchanged` as its first target. The series adds core infrastructure (`you_still_use_that()` helper) and modifies `git whatchanged` to require `--i-still-use-this` while pointing users toward `git log --raw`. Junio Hamano clarified this initial phase is meant to gauge usage rather than announce formal deprecation, following the successful pattern used for `pack-redundant`. Discussion continues about whether deprecation warnings should include suggested alternatives.

**Windows/Meson build assertion handling** -- Johannes Schindelin defended his patch configuring Windows/Meson builds to use release mode, explaining this avoids modal dialogs from MSVC assertions and maintains consistency with long-standing Windows build practices. Junio Hamano accepted the technical justification after initially suggesting the patch could be dropped, indicating it may be fast-tracked to the maintenance branch. This resolves a thread that began with a `git mv` bugfix and expanded into broader discussion about assertion handling policies.

## In brief

**Bash function recognition improvements** -- Final patch in a series improving Git's Bash function recognition awaits maintainer approval, simplifying detection while maintaining backward compatibility.

**Git 20th anniversary interviews** -- Kaartic Sivaraam shared the compiled draft of community interviews for the special anniversary edition of Git Rev News, now in final review.

**Promisor-remote protocol enhancement** -- Christian Couder confirmed case sensitivity handling in the promisor-remote protocol has been properly implemented in v2 of his series.

**Bitmap lookup tables status check** -- Junio Hamano inquired about the status of Taylor Blau's series to enable bitmap lookup tables by default, noting limited reviewer engagement.

**Meson shell path detection finalized** -- The thread about shell path detection in Meson builds reached consensus that the `/bin/sh` preference with `paths.ini` configuration provides sufficient flexibility.

**`git apply --intent-to-add` bug** -- Ryan Hodges demonstrated concrete cases where the current implementation corrupts the index by incorrectly marking unrelated tracked files as deleted.

**Safe.directory behavior with SSH clones** -- A bug report highlighted issues with Git's safe.directory check failing for SSH clones where the remote repository has different group ownership.

**On the radar**

**Optional file handling design** -- Phillip Wood joined Patrick Steinhardt in questioning whether empty files should be treated identically to missing files in Junio Hamano's new `:(optional)` path handling mechanism.

**Diff context unification** -- Leon Michalak's effort to unify diff context handling between interactive and non-interactive commands advanced with discussion about sentinel values and validation layer placement.