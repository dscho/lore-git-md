Here's the daily digest for February 10, 2026:

**The day in brief.** A busy day with 98 emails across 29 threads, dominated by technical discussions around ref backend configuration, `the_repository` removal, and security hardening of `git am`. Key developments include Junio signaling readiness to merge the ref backend selection series and substantive progress on UTF-8 alias support.

**Notable threads**

**Ref backend selection nears completion**  
Karthik Nayak's series enabling configurable reference storage backends (files<->reftable) has reached its final polishing phase after addressing all substantive feedback. Junio Hamano indicates the changes look ready for the 'next' branch, pending only minor documentation tweaks from Patrick Steinhardt's review. The implementation supports both persistent (`extensions.refStorage`) and ephemeral (`GIT_REFERENCE_BACKEND`) control, primarily for GitLab's migration needs. Jeff King contributed last-minute const-correctness fixes to the URI parsing logic.

**Worktree marker inconsistency surfaces in global removal**  
A discussion in Shreyansh Paliwal's `the_repository` removal thread revealed an architectural inconsistency: the worktree subsystem uses NULL `id` to mark the main worktree while ref stores use a forward slash ('/'). Junio Hamano and Karthik Nayak agreed standardizing on the slash would be preferable, though it requires auditing all NULL worktree checks. This decision will affect multiple in-flight series including ref backend and reftable work.

**UTF-8 alias support refined**  
Jonatan Holmgren's series to allow UTF-8 in Git aliases via config subsections advanced to v3 after addressing extensive review feedback. The implementation now properly handles case sensitivity differences between traditional syntax (case-insensitive) and subsection syntax (case-sensitive), with comprehensive test coverage. Junio Hamano raised final questions about NULL value handling in the alias listing refactoring, but the series appears merge-ready once resolved.

**Security discussions continue for git am`**  
The thread about hardening `git am` against accidental patch application saw multiple approaches debated. Phillip Wood and Kristoffer Haugsbakk finalized documentation warnings, while Patrick Steinhardt and Jacob Keller explored more fundamental solutions like an "unambiguous mode" for patch generation. Junio Hamano suggested leveraging signature verification as an alternative security boundary, though no consensus emerged on code changes beyond the documentation improvements.

**In brief**  
**Ref backend test coverage** -- Karthik Nayak expands test cases for URI-based reference locations, addressing Patrick Steinhardt's feedback on worktree path handling and dry-run behavior.

**ODB flag handling** -- Karthik Nayak and Patrick Steinhardt discuss type safety improvements for object database flags, considering making the `HAS_OBJECT_*` enum non-anonymous.

**Xdiff memory optimizations** -- Junio moves forward with Phillip Wood's conservative xdiff refactoring to reduce memory usage, temporarily setting aside Elijah Newren's more aggressive approach.

**Config batch command RFC** -- Derrick Stolee pivots to improving `git config list --type` filtering after feedback showed his batch proposal was overly complex for most use cases.

**Pre-add hook proposal** -- A new `pre-add` hook RFC drew maintainer feedback that it runs too early in the process, with Junio suggesting a two-phase index design to provide more context.

**On the radar**  
**Submodule config access** -- Test failures in the `the_repository` removal effort reveal submodules incorrectly accessing config from non-primary repository instances, requiring architectural fixes.

**Iconv test handling** -- Patrick Steinhardt and Torsten Bögershausen continue discussing how to handle encoding tests when the `iconv(1)` binary is unavailable on Windows/MSVC platforms.