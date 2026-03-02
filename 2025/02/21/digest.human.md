Here's the daily digest for February 21, 2025:

## The day in brief
A busy day with 82 emails across 28 threads, featuring significant progress on several fronts. The standout developments include the completion of the remote object-info series, major performance optimizations for `git cat-file` filtering, and a breakthrough in bisection speed for large repositories. Meanwhile, platform-specific fixes and documentation refinements rounded out the day's activity.

## Notable threads

### Remote object-info implementation finalized
Peijian Ju submitted the final version (v11) of an 8-part series implementing client-side support for querying object information from remote servers via `git cat-file --batch-command`. The series, building on Calvin Wan's earlier work, introduces a new `remote-object-info` command that lets clients query object sizes without downloading full objects. The implementation includes comprehensive security hardening based on Jeff King's review, with protections against buffer overflows, integer overflows, and resource exhaustion. With all feedback addressed and tests passing, this polished implementation appears ready for merging.

### Cat-file filtering gets major performance boost
Patrick Steinhardt kicked off a 9-part series dramatically improving `git cat-file`'s batch filtering capabilities. The patches add type-based filtering (matching `git rev-list` syntax) and leverage bitmap indices to avoid full packfile scans, showing 4000x speedups for tag filtering in large repositories like Chromium (20ms vs 82s). The implementation introduces new filter types ("blob:none", "blob:limit=", "object:type=") while maintaining backward compatibility. The series culminates with bitmap optimizations that make type filtering scale with matching objects rather than repository size.

### Bisection optimization breakthrough
Junio Hamano presented an experimental patch that reduces `git bisect` initialization time in large repositories from 20+ minutes to ~30 seconds by implementing midpoint merge traversal. The optimization changes how merges are processed during bisection, prioritizing those near the middle of the commit list first. Christian Couder and others discussed follow-up optimizations like loosened halfway thresholds and first-parent bisection hybrids, though consensus emerged that the core optimization is ready for broader testing. The Linux kernel repository (572k+ commits) served as the benchmark case.

### Partial reference transactions land
Karthik Nayak's series enabling partial reference transactions reached its final form with the addition of a `--allow-partial` flag for `git update-ref --stdin`. The implementation allows some reference updates to fail while others succeed within a single transaction, addressing a long-standing limitation in Git's ref handling. The thread included detailed discussion about error classification (user-facing vs system errors) and backend behavior differences (files vs reftable), with the core functionality now ready for merging pending a prerequisite `git reflog drop` implementation.

### Promisor-remote/LOP series concludes
After extensive review, Patrick Steinhardt gave final approval to Christian Couder's promisor-remote advertisement series, which Junio has queued. The discussion turned to future enhancements like persisting advertised promisors client-side and authentication mechanisms, though these were explicitly separated from the current production-ready foundation. The series implements protocol v2 capability negotiation for large object promisors (LOP), completing a major piece of Git's partial clone infrastructure.

## In brief
Jeff King and Toon Claes refined progress reporting in bundle-URI HTTPS downloads, settling on a structured protocol over stdout for clean separation from stderr errors. Documentation improvements clarified submodule merge behavior, though debate continued about avoiding text between 'ort' and 'recursive' strategy sections. Phillip Wood fixed edge cases in `git add -p`'s hunk splitting behavior, ensuring proper state inheritance after splits and edits. A Windows-specific bug in `git mv` symlink handling was fixed by properly setting `FILE_FLAG_OPEN_REPARSE_POINT`. OpenBSD builds gained compatibility with clang 19's stricter warnings through `_XOPEN_SOURCE` adjustments. The refs subsystem saw performance optimizations for ambiguity checks and backend migrations, with `--no-reflog` support now awaiting prerequisite work.

## On the radar
The `show-branch` refactoring to remove `MAX_REVS` limitations is exploring use of the long-dormant `commit_slab_with_stride` infrastructure. Performance benchmarking discussions between GitLab and Google are converging on requirements for reproducible test cases and actionable reports. A user-reported `git diff` behavior during merge conflicts (omitting some changes until staging) may warrant investigation as it surfaces potentially interesting diff machinery behavior.