# Git Mailing List Digest - 2025/11/05

**The day in brief.** A busy Wednesday with 55 emails across 16 threads, dominated by several significant patch series reaching final approval. The atomic ref updates for `git replay` are now ready, Junio's incomplete-line whitespace series advances to v2, and `git rebase` gains trailer support. Meanwhile, discussions continue on signature stripping in fast-import and extended attribute reporting in diff.

## Notable threads

**Atomic ref updates for git replay reach merge-ready state**  
Siddharth Asthana's series implementing atomic reference updates in `git replay` has completed its review cycle with all maintainer feedback addressed. The v7 series introduces `--ref-action` (with update/print modes) and a matching `replay.refAction` config option, using Git's ref transaction API for atomic behavior. Elijah Newren, Christian Couder, and Phillip Wood contributed reviews focusing on naming consistency, test hygiene, and reflog messages. The implementation now properly handles bare repositories and includes comprehensive tests for the new atomic behavior. With Junio's approval, this production-ready feature is queued for merging.

**Junio's incomplete-line whitespace series advances to v2**  
Junio Hamano's 12-part series to detect missing terminating newlines as configurable whitespace errors has progressed to version 2. The implementation now properly handles line number reporting in `git apply --check` and adds extensive test coverage. The series follows a careful progression from preparatory refactoring (patches 1-8) through bitmask infrastructure (9) to actual implementation (10-12). Patch 12 enforces the check across Git's own codebase via `.gitattributes`. The v2 changes address edge cases around reverse patch application and error messaging while maintaining backward compatibility.

**Rebase gains in-process trailer support**  
A 4-part series from Kristoffer Haugsbakk adds `--trailer` support to `git rebase`, eliminating fork/exec overhead by moving trailer processing in-process. The implementation builds through careful refactoring: first extracting core trailer logic (1), then making it reusable (2), before finally adding rebase integration (4) with merge-backend-only support. Junio's review requests clearer documentation of the public `process_trailers()` API and suggests splitting some interface changes. The series includes 134 lines of new tests covering trailer validation and edge cases.

**In brief.**  
**HTTP protocol documentation finalized** -- Clarifies server behavior for invalid `want` lines, requiring problematic object names in error responses.  

**Reftable optimization checks** -- Karthik Nayak explains compaction logic reuse in the new `is-needed` subcommand series, addressing Junio's concerns about code duplication.  

**Git data model documentation debate** -- Julia Evans and Ben Knoble discuss pedagogical approaches to visualizing commit objects versus their names, with Junio emphasizing technical accuracy.  

**Delta pager bug identified** -- The `git grep -l` filename formatting issue is traced to a known Delta pager bug (#1259), not Git itself.  

**Windows credential helper updates** -- Makefile modernization for the wincred helper, aligning with current Git conventions.  

**On the radar.**  
**Signature stripping in fast-import** -- Junio raises concerns about cryptographic integrity when stripping invalid signatures during history rewriting, potentially affecting the proposed 'strip-if-invalid' mode.  

**Extended attribute reporting** -- The discussion on `git diff --raw` format extensions continues, weighing key=value pairs against simpler letter codes for script parsing.