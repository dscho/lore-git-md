Here's the daily digest for January 5, 2026:

**The day in brief.** A busy day with 89 emails across 36 threads, featuring major progress on several fronts. The hook subsystem refactoring received final approvals for merging into `next`, while significant work continued on ref backend selection, object info handling, and geometric repacking. Documentation improvements and test modernizations dominated the volume work, alongside several bug reports and new contributor discussions.

**Notable threads**

**Hook subsystem standardization complete**  
Adrian Ratiu's multi-year effort to standardize Git's hook infrastructure has received final administrative confirmation for merging into `next`. The 13-part v6 series converts all major hooks to use the new `hook.h` API, introducing structured stdin handling, output capture, and parallel execution capabilities while maintaining backward compatibility. With Junio Hamano's approval ("I didn't see anything iffy") and Patrick Steinhardt's sign-off on the final iteration, this architectural improvement is now cleared for integration after extensive review.

**Ref backend selection nears completion**  
The ref storage backend selection feature is addressing its final major piece - worktree integration. Karthik Nayak and Patrick Steinhardt discussed how worktree ref stores should follow the main repository's configured storage location when using the new `extension.refStorage` config. The thread has settled on a URI-based configuration syntax with environment variable override capability (`GIT_REFERENCE_BACKEND`), with worktree support being the last substantive implementation question before merging.

**Object info handling refined**  
Patrick Steinhardt's object info refactoring series received detailed review on its delta type handling, with Toon Claes questioning whether distinguishing between ref-delta and ofs-delta is necessary at the interface level. The discussion revealed careful attention to the API design as the series moves toward standardizing how Git reports object metadata, particularly from packfiles. A minor correction was also acknowledged for a commit message typo ("inconsistecny" -> "inconsistency") in patch 4/7.

**`git status` push tracking finalized**  
Harald Nordgren's 17-iteration series to show push remote divergence in `git status` output has reached its final form. The implementation now cleanly handles custom refspec transformations and includes comprehensive test coverage. Junio Hamano provided last-minute feedback about control flow in the branch comparison logic, confirming the correct evaluation order for divergence checks. The change represents a significant UX improvement for users working with multiple remotes.

**Geometric repacking with promisor remotes**  
Patrick Steinhardt sent a 5-patch series fixing incompatibilities between geometric repacking and promisor remotes. The changes allow combining `--stdin-packs` with `--exclude-promisor-objects` in `pack-objects`, refactor the geometric calculation logic for reuse, and implement proper promisor pack handling during repacks. The series demonstrates careful incremental refactoring to support this advanced use case while maintaining existing behavior.

**In brief**  
**`git reset` documentation** -- D. Ben Knoble shepherds final v4 polish to the man page improvements, adding backticks and imperative mood per Jean-Noël Avila's feedback.

**Lockfile PID debugging** -- Patrick Steinhardt agrees with Jeff King's proposal to simplify the configuration interface from per-component toggles to a single boolean.

**`git replay` error handling** -- Kristoffer Haugsbakk's v3 series improves error messages for invalid commit-ish inputs with comprehensive test coverage.

**Test modernization** -- Multiple contributors advanced the effort to replace shell tests with helper functions in t1300, t2021, and t5403.

**`git last-modified` behavior** -- Toon Claes addressed performance characteristics and directory handling in this new command, with a crash bug identified in refspec parsing.

**`git whatchanged` deprecation** -- Users confirmed `git log --raw` serves as an adequate replacement, validating the deprecation path.

**On the radar**  
**`the_repository` removal** -- Phillip Wood identified a UX regression in lazy config parsing that may affect other settings migrations.

**Bazel build proposal** -- Initial discussion shows skepticism about adding a fourth build system without addressing configurability concerns.

**`git diff --relative` behavior** -- A bug report suggests unexpected filename processing that may need documentation clarification or code changes.